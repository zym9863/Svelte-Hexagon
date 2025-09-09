<script lang="ts">
  import HexagonBall from './lib/HexagonBall.svelte'
</script>

<main>
  <div class="app-container">
    <header class="app-header">
      <h1>旋转六边形弹跳球</h1>
      <p class="subtitle">物理引擎模拟 - 重力、摩擦力、碰撞检测</p>
    </header>
    
    <div class="game-container">
      <HexagonBall 
        width={600} 
        height={600} 
        hexagonRadius={200}
      />
    </div>
    
    <div class="instructions">
      <h3>使用说明：</h3>
      <ul>
        <li>点击"开始"按钮启动物理模拟</li>
        <li>点击"暂停"按钮停止动画</li>
        <li>点击"重置"按钮重新开始</li>
        <li>调整滑块来改变物理参数</li>
        <li>观察小球在旋转六边形内的弹跳行为</li>
      </ul>
    </div>
  </div>
</main>

<style>
  .app-container {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 2rem;
    background: linear-gradient(135deg, 
      #0f0f23 0%, 
      #1a1a2e 25%, 
      #16213e 50%, 
      #1a1a2e 75%, 
      #0f0f23 100%);
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    position: relative;
    overflow-x: hidden;
  }

  .app-container::before {
    content: '';
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: 
      radial-gradient(circle at 20% 20%, rgba(0, 255, 136, 0.1) 0%, transparent 50%),
      radial-gradient(circle at 80% 80%, rgba(0, 204, 255, 0.1) 0%, transparent 50%),
      radial-gradient(circle at 40% 60%, rgba(255, 107, 107, 0.05) 0%, transparent 50%);
    pointer-events: none;
    z-index: -1;
    animation: floatingOrbs 20s ease-in-out infinite;
  }

  @keyframes floatingOrbs {
    0%, 100% { 
      background: 
        radial-gradient(circle at 20% 20%, rgba(0, 255, 136, 0.1) 0%, transparent 50%),
        radial-gradient(circle at 80% 80%, rgba(0, 204, 255, 0.1) 0%, transparent 50%),
        radial-gradient(circle at 40% 60%, rgba(255, 107, 107, 0.05) 0%, transparent 50%);
    }
    33% { 
      background: 
        radial-gradient(circle at 80% 30%, rgba(0, 255, 136, 0.1) 0%, transparent 50%),
        radial-gradient(circle at 20% 70%, rgba(0, 204, 255, 0.1) 0%, transparent 50%),
        radial-gradient(circle at 60% 20%, rgba(255, 107, 107, 0.05) 0%, transparent 50%);
    }
    66% { 
      background: 
        radial-gradient(circle at 40% 80%, rgba(0, 255, 136, 0.1) 0%, transparent 50%),
        radial-gradient(circle at 70% 20%, rgba(0, 204, 255, 0.1) 0%, transparent 50%),
        radial-gradient(circle at 20% 40%, rgba(255, 107, 107, 0.05) 0%, transparent 50%);
    }
  }

  .app-header {
    text-align: center;
    margin-bottom: 2.5rem;
    color: white;
    position: relative;
  }

  .app-header h1 {
    font-size: 3rem;
    margin: 0 0 0.5rem 0;
    background: linear-gradient(45deg, #00ff88, #00ccff, #ff6b6b, #00ff88);
    background-size: 300% 300%;
    animation: gradientShift 6s ease-in-out infinite;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    font-weight: 800;
    text-shadow: 0 0 30px rgba(0, 255, 136, 0.3);
    letter-spacing: 1px;
  }

  @keyframes gradientShift {
    0%, 100% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
  }

  .subtitle {
    font-size: 1.2rem;
    opacity: 0.9;
    margin: 0;
    color: #b8b8d9;
    font-weight: 300;
    letter-spacing: 0.5px;
  }

  .game-container {
    margin-bottom: 2.5rem;
    border-radius: 20px;
    overflow: hidden;
    box-shadow: 
      0 25px 50px rgba(0, 0, 0, 0.5),
      0 0 50px rgba(0, 255, 136, 0.1);
    position: relative;
  }

  .game-container::before {
    content: '';
    position: absolute;
    top: -2px;
    left: -2px;
    right: -2px;
    bottom: -2px;
    background: linear-gradient(45deg, #00ff88, #00ccff, #ff6b6b, #00ff88);
    background-size: 300% 300%;
    animation: gradientShift 6s ease-in-out infinite;
    border-radius: 22px;
    z-index: -1;
  }

  .instructions {
    background: linear-gradient(135deg, 
      rgba(255, 255, 255, 0.1) 0%, 
      rgba(255, 255, 255, 0.05) 100%);
    backdrop-filter: blur(15px);
    border-radius: 16px;
    padding: 2rem;
    color: white;
    max-width: 700px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    position: relative;
    overflow: hidden;
  }

  .instructions::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 1px;
    background: linear-gradient(90deg, transparent, rgba(0, 255, 136, 0.5), transparent);
  }

  .instructions h3 {
    margin-top: 0;
    color: #00ff88;
    font-size: 1.3rem;
    font-weight: 700;
    margin-bottom: 1rem;
    text-transform: uppercase;
    letter-spacing: 1px;
  }

  .instructions ul {
    margin: 0;
    padding-left: 1.5rem;
  }

  .instructions li {
    margin-bottom: 0.75rem;
    line-height: 1.6;
    color: rgba(255, 255, 255, 0.9);
    transition: color 0.2s ease;
  }

  .instructions li:hover {
    color: #00ccff;
  }

  @media (max-width: 768px) {
    .app-container {
      padding: 1rem;
    }
    
    .app-header h1 {
      font-size: 2.2rem;
    }
    
    .subtitle {
      font-size: 1rem;
    }
    
    .game-container {
      transform: scale(0.85);
      transform-origin: center;
    }

    .instructions {
      padding: 1.5rem;
    }
  }

  @media (max-width: 480px) {
    .app-header h1 {
      font-size: 1.8rem;
    }
    
    .game-container {
      transform: scale(0.7);
    }

    .instructions {
      padding: 1rem;
    }
  }
</style>
