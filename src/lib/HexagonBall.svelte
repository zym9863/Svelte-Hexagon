<script lang="ts">
  import { onMount, onDestroy } from 'svelte';

  interface BallState {
    x: number;
    y: number;
    vx: number;
    vy: number;
    radius: number;
  }

  interface PhysicsParams {
    gravity: number;
    friction: number;
    restitution: number;
    rotationSpeed: number;
  }

  interface Point {
    x: number;
    y: number;
  }

  interface ParticleTrail {
    x: number;
    y: number;
    life: number;
    maxLife: number;
    size: number;
  }

  // Component props
  export let width = 600;
  export let height = 600;
  export let hexagonRadius = 200;
  export let physics: PhysicsParams = {
    gravity: 0.3,
    friction: 0.995,
    restitution: 0.85,
    rotationSpeed: 0.8
  };

  // DOM references
  let canvas: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;
  let animationId: number;
  let containerEl: HTMLDivElement;

  // State
  let ball: BallState;
  let rotationAngle = 0;
  let isRunning = true;
  let hexagonVertices: Point[] = [];
  let particleTrails: ParticleTrail[] = [];
  let collisionGlow = 0;

  // Initialize ball state
  function initBall(): BallState {
    return {
      x: width / 2,
      y: height / 2 - 80,
      vx: 2,
      vy: 0,
      radius: 12
    };
  }

  // Calculate hexagon vertices based on rotation angle
  function calculateHexagonVertices(angle: number): Point[] {
    const centerX = width / 2;
    const centerY = height / 2;
    const vertices: Point[] = [];
    
    for (let i = 0; i < 6; i++) {
      const vertexAngle = (i * Math.PI / 3) + (angle * Math.PI / 180);
      vertices.push({
        x: centerX + hexagonRadius * Math.cos(vertexAngle),
        y: centerY + hexagonRadius * Math.sin(vertexAngle)
      });
    }
    
    return vertices;
  }

  // Calculate distance from point to line segment
  function pointToLineDistance(point: Point, lineStart: Point, lineEnd: Point): number {
    const A = point.x - lineStart.x;
    const B = point.y - lineStart.y;
    const C = lineEnd.x - lineStart.x;
    const D = lineEnd.y - lineStart.y;

    const dot = A * C + B * D;
    const lenSq = C * C + D * D;
    
    if (lenSq === 0) return Math.sqrt(A * A + B * B);
    
    let param = dot / lenSq;
    param = Math.max(0, Math.min(1, param));
    
    const xx = lineStart.x + param * C;
    const yy = lineStart.y + param * D;
    
    const dx = point.x - xx;
    const dy = point.y - yy;
    
    return Math.sqrt(dx * dx + dy * dy);
  }

  // Calculate line normal vector
  function getLineNormal(lineStart: Point, lineEnd: Point): Point {
    const dx = lineEnd.x - lineStart.x;
    const dy = lineEnd.y - lineStart.y;
    const length = Math.sqrt(dx * dx + dy * dy);
    
    return {
      x: -dy / length,
      y: dx / length
    };
  }

  // Add particle trail
  function addParticleTrail() {
    const speed = Math.sqrt(ball.vx * ball.vx + ball.vy * ball.vy);
    if (speed > 0.5) {
      particleTrails.push({
        x: ball.x,
        y: ball.y,
        life: 30,
        maxLife: 30,
        size: Math.min(6, speed * 2)
      });
    }
    
    // Limit number of particles for performance
    if (particleTrails.length > 20) {
      particleTrails.shift();
    }
  }

  // Update particle trails
  function updateParticleTrails() {
    for (let i = particleTrails.length - 1; i >= 0; i--) {
      particleTrails[i].life--;
      if (particleTrails[i].life <= 0) {
        particleTrails.splice(i, 1);
      }
    }
  }

  // Check collision with hexagon walls and handle bounce
  function checkCollision(): boolean {
    for (let i = 0; i < hexagonVertices.length; i++) {
      const start = hexagonVertices[i];
      const end = hexagonVertices[(i + 1) % hexagonVertices.length];
      
      const distance = pointToLineDistance(ball, start, end);
      
      if (distance <= ball.radius) {
        // Calculate normal vector
        const normal = getLineNormal(start, end);
        
        // Move ball outside the wall
        const overlap = ball.radius - distance;
        ball.x += normal.x * overlap;
        ball.y += normal.y * overlap;
        
        // Reflect velocity
        const dotProduct = ball.vx * normal.x + ball.vy * normal.y;
        ball.vx -= 2 * dotProduct * normal.x;
        ball.vy -= 2 * dotProduct * normal.y;
        
        // Apply restitution (energy loss)
        ball.vx *= physics.restitution;
        ball.vy *= physics.restitution;
        
        // Add collision glow effect
        collisionGlow = 15;
        
        return true;
      }
    }
    return false;
  }

  // Update physics
  function updatePhysics() {
    // Apply gravity
    ball.vy += physics.gravity;
    
    // Apply friction
    ball.vx *= physics.friction;
    ball.vy *= physics.friction;
    
    // Update position
    ball.x += ball.vx;
    ball.y += ball.vy;
    
    // Add particle trail
    addParticleTrail();
    
    // Update particle trails
    updateParticleTrails();
    
    // Reduce collision glow
    if (collisionGlow > 0) {
      collisionGlow--;
    }
    
    // Check collisions
    checkCollision();
    
    // Update rotation
    rotationAngle += physics.rotationSpeed;
    hexagonVertices = calculateHexagonVertices(rotationAngle);
  }

  // Render function
  function render() {
    if (!ctx) return;
    
    // Clear canvas with subtle background
    ctx.fillStyle = 'rgba(15, 15, 35, 0.1)';
    ctx.fillRect(0, 0, width, height);
    
    // Draw particle trails first
    for (let i = 0; i < particleTrails.length; i++) {
      const particle = particleTrails[i];
      const alpha = particle.life / particle.maxLife;
      const size = particle.size * alpha;
      
      const gradient = ctx.createRadialGradient(
        particle.x, particle.y, 0,
        particle.x, particle.y, size
      );
      gradient.addColorStop(0, `rgba(255, 135, 135, ${alpha * 0.8})`);
      gradient.addColorStop(1, `rgba(255, 107, 107, 0)`);
      
      ctx.fillStyle = gradient;
      ctx.beginPath();
      ctx.arc(particle.x, particle.y, size, 0, Math.PI * 2);
      ctx.fill();
    }
    
    // Draw enhanced hexagon with dynamic glow
    const hexagonGlow = 10 + collisionGlow;
    ctx.shadowColor = `rgba(0, 255, 136, ${0.6 + collisionGlow * 0.03})`;
    ctx.shadowBlur = hexagonGlow;
    
    // Create hexagon gradient
    const hexagonGradient = ctx.createLinearGradient(
      width / 2 - hexagonRadius, height / 2 - hexagonRadius,
      width / 2 + hexagonRadius, height / 2 + hexagonRadius
    );
    hexagonGradient.addColorStop(0, '#00ff88');
    hexagonGradient.addColorStop(0.5, '#00ccff');
    hexagonGradient.addColorStop(1, '#00ff88');
    
    ctx.strokeStyle = hexagonGradient;
    ctx.lineWidth = 3;
    ctx.beginPath();
    
    if (hexagonVertices.length > 0) {
      ctx.moveTo(hexagonVertices[0].x, hexagonVertices[0].y);
      for (let i = 1; i < hexagonVertices.length; i++) {
        ctx.lineTo(hexagonVertices[i].x, hexagonVertices[i].y);
      }
      ctx.closePath();
    }
    
    ctx.stroke();
    ctx.shadowBlur = 0; // Reset shadow
    
    // Draw ball with enhanced gradient and dynamic glow
    const ballSpeed = Math.sqrt(ball.vx * ball.vx + ball.vy * ball.vy);
    const ballGlow = Math.min(20, 5 + ballSpeed * 2 + collisionGlow);
    
    ctx.shadowColor = `rgba(255, 107, 107, ${0.6 + collisionGlow * 0.05})`;
    ctx.shadowBlur = ballGlow;
    
    const gradient = ctx.createRadialGradient(
      ball.x - ball.radius * 0.3,
      ball.y - ball.radius * 0.3,
      0,
      ball.x,
      ball.y,
      ball.radius * 1.5
    );
    
    // Dynamic ball colors based on speed and collision
    const baseR = 255;
    const baseG = Math.max(100, 135 - ballSpeed * 10);
    const baseB = Math.max(100, 135 - ballSpeed * 10 + collisionGlow * 5);
    
    gradient.addColorStop(0, `rgb(${baseR}, ${baseG + 50}, ${baseB + 50})`);
    gradient.addColorStop(0.4, `rgb(${baseR}, ${baseG}, ${baseB})`);
    gradient.addColorStop(0.8, `rgb(${Math.max(150, baseR - 50)}, ${Math.max(50, baseG - 50)}, ${Math.max(50, baseB - 50)})`);
    gradient.addColorStop(1, `rgb(${Math.max(100, baseR - 100)}, ${Math.max(20, baseG - 80)}, ${Math.max(20, baseB - 80)})`);
    
    ctx.fillStyle = gradient;
    ctx.beginPath();
    ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
    ctx.fill();
    
    ctx.shadowBlur = 0; // Reset shadow
    
    // Add enhanced ball highlight
    const highlightGradient = ctx.createRadialGradient(
      ball.x - ball.radius * 0.4,
      ball.y - ball.radius * 0.4,
      0,
      ball.x - ball.radius * 0.4,
      ball.y - ball.radius * 0.4,
      ball.radius * 0.5
    );
    highlightGradient.addColorStop(0, 'rgba(255, 255, 255, 0.8)');
    highlightGradient.addColorStop(1, 'rgba(255, 255, 255, 0)');
    
    ctx.fillStyle = highlightGradient;
    ctx.beginPath();
    ctx.arc(
      ball.x - ball.radius * 0.3,
      ball.y - ball.radius * 0.3,
      ball.radius * 0.4,
      0,
      Math.PI * 2
    );
    ctx.fill();
  }

  // Animation loop
  function animate() {
    if (!isRunning) return;
    
    updatePhysics();
    render();
    
    animationId = requestAnimationFrame(animate);
  }

  // Control functions
  export function start() {
    isRunning = true;
    animate();
  }

  export function stop() {
    isRunning = false;
    if (animationId) {
      cancelAnimationFrame(animationId);
    }
  }

  export function reset() {
    stop();
    ball = initBall();
    rotationAngle = 0;
    particleTrails = [];
    collisionGlow = 0;
    hexagonVertices = calculateHexagonVertices(rotationAngle);
    render();
  }

  // Lifecycle
  onMount(() => {
    if (canvas) {
      const context = canvas.getContext('2d');
      if (context) {
        ctx = context;
        ball = initBall();
        hexagonVertices = calculateHexagonVertices(rotationAngle);
        render();
        start();
      }
    }
  });

  onDestroy(() => {
    stop();
  });
</script>

<div class="hexagon-ball-container" bind:this={containerEl}>
  <div class="controls">
    <button on:click={start}>开始</button>
    <button on:click={stop}>暂停</button>
    <button on:click={reset}>重置</button>
  </div>
  
  <div class="canvas-container">
    <canvas
      bind:this={canvas}
      {width}
      {height}
      class="game-canvas"
    ></canvas>
    
    <!-- SVG Hexagon for reference -->
    <svg 
      class="hexagon-svg" 
      {width} 
      {height} 
      viewBox="0 0 {width} {height}"
    >
      <g transform="translate({width/2}, {height/2})">
        <path
          d="M {hexagonRadius} 0 L {hexagonRadius * Math.cos(Math.PI/3)} {hexagonRadius * Math.sin(Math.PI/3)} L {hexagonRadius * Math.cos(2*Math.PI/3)} {hexagonRadius * Math.sin(2*Math.PI/3)} L {-hexagonRadius} 0 L {hexagonRadius * Math.cos(4*Math.PI/3)} {hexagonRadius * Math.sin(4*Math.PI/3)} L {hexagonRadius * Math.cos(5*Math.PI/3)} {hexagonRadius * Math.sin(5*Math.PI/3)} Z"
          fill="none"
          stroke="#00ff88"
          stroke-width="2"
          opacity="0.3"
          transform="rotate({rotationAngle})"
        />
      </g>
    </svg>
  </div>
  
  <div class="physics-controls">
    <div class="control-group">
      <label for="gravity-control">重力强度:</label>
      <input 
        id="gravity-control"
        type="range" 
        min="0.1" 
        max="1" 
        step="0.1" 
        bind:value={physics.gravity}
      />
      <span>{physics.gravity.toFixed(1)}</span>
    </div>
    
    <div class="control-group">
      <label for="rotation-control">旋转速度:</label>
      <input 
        id="rotation-control"
        type="range" 
        min="0" 
        max="3" 
        step="0.1" 
        bind:value={physics.rotationSpeed}
      />
      <span>{physics.rotationSpeed.toFixed(1)}</span>
    </div>
    
    <div class="control-group">
      <label for="friction-control">摩擦系数:</label>
      <input 
        id="friction-control"
        type="range" 
        min="0.9" 
        max="1" 
        step="0.01" 
        bind:value={physics.friction}
      />
      <span>{physics.friction.toFixed(2)}</span>
    </div>
    
    <div class="control-group">
      <label for="restitution-control">弹性系数:</label>
      <input 
        id="restitution-control"
        type="range" 
        min="0.5" 
        max="1" 
        step="0.05" 
        bind:value={physics.restitution}
      />
      <span>{physics.restitution.toFixed(2)}</span>
    </div>
  </div>
</div>

<style>
  .hexagon-ball-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
    padding: 1rem;
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
    border-radius: 12px;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  }

  .controls {
    display: flex;
    gap: 0.8rem;
    margin-bottom: 1.5rem;
    justify-content: center;
  }

  .controls button {
    padding: 0.75rem 1.5rem;
    background: linear-gradient(135deg, #00ff88 0%, #00ccff 100%);
    color: #1a1a2e;
    border: none;
    border-radius: 12px;
    font-weight: 700;
    cursor: pointer;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    font-size: 0.9rem;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    box-shadow: 0 4px 15px rgba(0, 255, 136, 0.3);
    position: relative;
    overflow: hidden;
  }

  .controls button::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
    transition: left 0.5s ease;
  }

  .controls button:hover {
    background: linear-gradient(135deg, #00ccff 0%, #00ff88 100%);
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(0, 255, 136, 0.4);
  }

  .controls button:hover::before {
    left: 100%;
  }

  .controls button:active {
    transform: translateY(0);
    box-shadow: 0 2px 10px rgba(0, 255, 136, 0.3);
  }

  .canvas-container {
    position: relative;
    border: 3px solid transparent;
    background: linear-gradient(135deg, #1a1a2e, #0f0f23) padding-box, 
                linear-gradient(135deg, #00ff88, #00ccff, #00ff88) border-box;
    border-radius: 16px;
    overflow: hidden;
    box-shadow: 
      0 0 30px rgba(0, 255, 136, 0.3),
      inset 0 0 30px rgba(0, 0, 0, 0.5);
  }

  .game-canvas {
    display: block;
    background: radial-gradient(circle at 30% 30%, #1a1a2e 0%, #16213e 50%, #0f0f23 100%);
  }

  .hexagon-svg {
    position: absolute;
    top: 0;
    left: 0;
    pointer-events: none;
  }

  .physics-controls {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
    gap: 1rem;
    width: 100%;
    max-width: 700px;
    margin-top: 1.5rem;
  }

  .control-group {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    background: linear-gradient(135deg, rgba(255, 255, 255, 0.08) 0%, rgba(255, 255, 255, 0.02) 100%);
    backdrop-filter: blur(10px);
    padding: 1rem;
    border-radius: 12px;
    border: 1px solid rgba(0, 255, 136, 0.2);
    transition: all 0.3s ease;
    position: relative;
    overflow: hidden;
  }

  .control-group::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 1px;
    background: linear-gradient(90deg, transparent, rgba(0, 255, 136, 0.5), transparent);
  }

  .control-group:hover {
    border-color: rgba(0, 255, 136, 0.4);
    background: linear-gradient(135deg, rgba(255, 255, 255, 0.12) 0%, rgba(255, 255, 255, 0.04) 100%);
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(0, 255, 136, 0.1);
  }

  .control-group label {
    color: #00ff88;
    font-weight: 600;
    font-size: 0.9rem;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }

  .control-group input[type="range"] {
    width: 100%;
    height: 6px;
    background: rgba(0, 255, 136, 0.2);
    border-radius: 3px;
    outline: none;
    appearance: none;
    -webkit-appearance: none;
    position: relative;
  }

  .control-group input[type="range"]::-webkit-slider-track {
    height: 6px;
    background: rgba(0, 255, 136, 0.2);
    border-radius: 3px;
  }

  .control-group input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 20px;
    height: 20px;
    background: linear-gradient(135deg, #00ff88, #00ccff);
    border-radius: 50%;
    cursor: pointer;
    box-shadow: 0 2px 10px rgba(0, 255, 136, 0.4);
    transition: all 0.2s ease;
  }

  .control-group input[type="range"]::-webkit-slider-thumb:hover {
    transform: scale(1.1);
    box-shadow: 0 4px 15px rgba(0, 255, 136, 0.6);
  }

  .control-group input[type="range"]::-moz-range-thumb {
    width: 20px;
    height: 20px;
    background: linear-gradient(135deg, #00ff88, #00ccff);
    border: none;
    border-radius: 50%;
    cursor: pointer;
    box-shadow: 0 2px 10px rgba(0, 255, 136, 0.4);
  }

  .control-group span {
    color: #ffffff;
    font-size: 0.85rem;
    text-align: center;
    font-weight: 700;
    background: rgba(0, 255, 136, 0.1);
    padding: 0.25rem 0.5rem;
    border-radius: 6px;
    border: 1px solid rgba(0, 255, 136, 0.2);
  }

  @media (max-width: 768px) {
    .hexagon-ball-container {
      padding: 0.5rem;
    }
    
    .physics-controls {
      grid-template-columns: 1fr;
    }
  }
</style>