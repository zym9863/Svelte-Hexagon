# 🎯 旋转六边形弹跳球 - 物理引擎模拟

**[English Version](./README-EN.md)** | **[中文版](./README.md)**

一个基于 Svelte + TypeScript + Vite 的高性能物理模拟项目，展示了一个在旋转六边形内弹跳的小球，具有真实的重力、摩擦力和碰撞效果。

## ✨ 功能特性

### 🎮 核心物理引擎
- **重力系统**: 可调节的重力加速度
- **摩擦力**: 空气阻力和表面摩擦
- **弹性碰撞**: 能量守恒与动量传递
- **旋转动力学**: 连续旋转的六边形边界

### 🎨 视觉效果
- **Canvas 渲染**: 流畅的 60fps 动画
- **渐变球体**: 3D 视觉效果的球体渲染
- **发光六边形**: 动态旋转的六边形边界
- **深色主题**: 现代化的科技风格界面

### 🎛 交互控制
- **实时控制**: 开始/暂停/重置功能
- **参数调节**: 重力、旋转速度、摩擦系数、弹性系数
- **响应式设计**: 适配移动端和桌面端

## 🚀 快速开始

### 安装依赖
```bash
pnpm install
```

### 开发服务器
```bash
pnpm dev
```
访问 `http://localhost:5173/` 查看应用

### 构建项目
```bash
pnpm build
```

### 预览构建结果
```bash
pnpm preview
```

## 🛠 技术栈

- **Svelte 5**: 现代化组件框架
- **TypeScript**: 类型安全的开发体验
- **Vite**: 快速的构建工具
- **Canvas API**: 高性能图形渲染
- **pnpm**: 高效的包管理器

## 📁 项目结构

```
src/
├── lib/
│   └── HexagonBall.svelte    # 主要的物理模拟组件
├── App.svelte                # 应用主组件
├── app.css                   # 全局样式
└── main.ts                   # 应用入口
```

## ⚙️ 物理参数说明

| 参数 | 范围 | 说明 |
|------|------|------|
| 重力强度 | 0.1 - 1.0 | 控制小球下落加速度 |
| 旋转速度 | 0 - 3 | 六边形旋转速度（度/帧）|
| 摩擦系数 | 0.9 - 1.0 | 速度衰减因子 |
| 弹性系数 | 0.5 - 1.0 | 碰撞后的能量保持率 |

## 🔧 核心算法

### 碰撞检测
使用线圆相交算法检测小球与六边形边的碰撞：
```typescript
function pointToLineDistance(point: Point, lineStart: Point, lineEnd: Point): number
```

### 物理更新
```typescript
function updatePhysics() {
  // 应用重力
  ball.vy += physics.gravity;
  
  // 应用摩擦力
  ball.vx *= physics.friction;
  ball.vy *= physics.friction;
  
  // 更新位置
  ball.x += ball.vx;
  ball.y += ball.vy;
  
  // 检测碰撞
  checkCollision();
}
```

## 🎨 视觉设计

- **深色科技主题**: 渐变背景配合霓虹色彩
- **发光效果**: CSS 阴影和光晕增强视觉层次
- **响应式布局**: 自适应不同屏幕尺寸
- **平滑动画**: requestAnimationFrame 确保流畅体验

## 📱 响应式特性

- 桌面端: 完整功能展示
- 平板端: 优化触控体验
- 手机端: 缩放适配小屏幕

## 🔍 性能优化

- **Canvas 渲染**: 避免 DOM 操作，提升渲染性能
- **requestAnimationFrame**: 与浏览器刷新率同步
- **TypeScript**: 编译时类型检查，减少运行时错误
- **组件化设计**: 模块化代码结构，易于维护

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request 来改进这个项目！

## 📄 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件

---

**享受物理模拟的乐趣！** 🎯✨
