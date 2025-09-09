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
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }

  .app-header {
    text-align: center;
    margin-bottom: 2rem;
    color: white;
  }

  .app-header h1 {
    font-size: 2.5rem;
    margin: 0 0 0.5rem 0;
    background: linear-gradient(45deg, #00ff88, #00ccff);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }

  .subtitle {
    font-size: 1.1rem;
    opacity: 0.9;
    margin: 0;
  }

  .game-container {
    margin-bottom: 2rem;
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
  }

  .instructions {
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    border-radius: 12px;
    padding: 1.5rem;
    color: white;
    max-width: 600px;
    border: 1px solid rgba(255, 255, 255, 0.2);
  }

  .instructions h3 {
    margin-top: 0;
    color: #00ff88;
  }

  .instructions ul {
    margin: 0;
    padding-left: 1.5rem;
  }

  .instructions li {
    margin-bottom: 0.5rem;
    line-height: 1.5;
  }

  @media (max-width: 768px) {
    .app-container {
      padding: 1rem;
    }
    
    .app-header h1 {
      font-size: 2rem;
    }
    
    .subtitle {
      font-size: 1rem;
    }
    
    .game-container {
      transform: scale(0.8);
      transform-origin: center;
    }
  }

  @media (max-width: 480px) {
    .game-container {
      transform: scale(0.6);
    }
  }
</style>
