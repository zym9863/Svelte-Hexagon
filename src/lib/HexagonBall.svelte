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
    
    // Check collisions
    checkCollision();
    
    // Update rotation
    rotationAngle += physics.rotationSpeed;
    hexagonVertices = calculateHexagonVertices(rotationAngle);
  }

  // Render function
  function render() {
    if (!ctx) return;
    
    // Clear canvas
    ctx.clearRect(0, 0, width, height);
    
    // Draw hexagon with glow effect
    ctx.shadowColor = '#00ff88';
    ctx.shadowBlur = 10;
    ctx.strokeStyle = '#00ff88';
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
    
    // Draw ball with enhanced gradient
    const gradient = ctx.createRadialGradient(
      ball.x - ball.radius * 0.4,
      ball.y - ball.radius * 0.4,
      0,
      ball.x,
      ball.y,
      ball.radius
    );
    gradient.addColorStop(0, '#ff8787');
    gradient.addColorStop(0.7, '#ff6b6b');
    gradient.addColorStop(1, '#c92a2a');
    
    ctx.fillStyle = gradient;
    ctx.beginPath();
    ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
    ctx.fill();
    
    // Add ball highlight
    ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
    ctx.beginPath();
    ctx.arc(
      ball.x - ball.radius * 0.3,
      ball.y - ball.radius * 0.3,
      ball.radius * 0.25,
      0,
      Math.PI * 2
    );
    ctx.fill();
    
    // Add subtle shadow
    ctx.fillStyle = 'rgba(0, 0, 0, 0.2)';
    ctx.beginPath();
    ctx.arc(
      ball.x + ball.radius * 0.2,
      ball.y + ball.radius * 0.2,
      ball.radius * 0.3,
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
    gap: 0.5rem;
    margin-bottom: 1rem;
  }

  .controls button {
    padding: 0.5rem 1rem;
    background: #00ff88;
    color: #1a1a2e;
    border: none;
    border-radius: 6px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .controls button:hover {
    background: #00cc6a;
    transform: translateY(-1px);
  }

  .canvas-container {
    position: relative;
    border: 2px solid #00ff88;
    border-radius: 8px;
    overflow: hidden;
    background: #0f0f23;
  }

  .game-canvas {
    display: block;
    background: radial-gradient(circle at center, #1a1a2e 0%, #0f0f23 100%);
  }

  .hexagon-svg {
    position: absolute;
    top: 0;
    left: 0;
    pointer-events: none;
  }

  .physics-controls {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1rem;
    width: 100%;
    max-width: 600px;
    margin-top: 1rem;
  }

  .control-group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    background: rgba(255, 255, 255, 0.05);
    padding: 0.75rem;
    border-radius: 6px;
    border: 1px solid rgba(0, 255, 136, 0.2);
  }

  .control-group label {
    color: #00ff88;
    font-weight: 500;
    font-size: 0.9rem;
  }

  .control-group input[type="range"] {
    width: 100%;
    height: 4px;
    background: rgba(0, 255, 136, 0.3);
    border-radius: 2px;
    outline: none;
    appearance: none;
    -webkit-appearance: none;
  }

  .control-group input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 16px;
    height: 16px;
    background: #00ff88;
    border-radius: 50%;
    cursor: pointer;
  }

  .control-group span {
    color: #ffffff;
    font-size: 0.8rem;
    text-align: center;
    font-weight: 600;
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