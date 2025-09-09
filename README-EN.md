# 🎯 Rotating Hexagon Bouncing Ball - Physics Engine Simulation

**[English Version](./README-EN.md)** | **[中文版](./README.md)**

A high-performance physics simulation project based on Svelte + TypeScript + Vite, featuring a bouncing ball inside a rotating hexagon with realistic gravity, friction, and collision effects.

## ✨ Features

### 🎮 Core Physics Engine
- **Gravity System**: Adjustable gravity acceleration
- **Friction**: Air resistance and surface friction
- **Elastic Collision**: Energy conservation and momentum transfer
- **Rotational Dynamics**: Continuously rotating hexagon boundary

### 🎨 Visual Effects
- **Canvas Rendering**: Smooth 60fps animation
- **Gradient Sphere**: 3D visual effect ball rendering
- **Glowing Hexagon**: Dynamically rotating hexagon boundary
- **Dark Theme**: Modern tech-style interface

### 🎛 Interactive Controls
- **Real-time Control**: Start/pause/reset functionality
- **Parameter Adjustment**: Gravity, rotation speed, friction coefficient, elasticity coefficient
- **Responsive Design**: Adapted for mobile and desktop

## 🚀 Quick Start

### Install Dependencies
```bash
pnpm install
```

### Development Server
```bash
pnpm dev
```
Visit `http://localhost:5173/` to view the application

### Build Project
```bash
pnpm build
```

### Preview Build Result
```bash
pnpm preview
```

## 🛠 Technology Stack

- **Svelte 5**: Modern component framework
- **TypeScript**: Type-safe development experience
- **Vite**: Fast build tool
- **Canvas API**: High-performance graphics rendering
- **pnpm**: Efficient package manager

## 📁 Project Structure

```
src/
├── lib/
│   └── HexagonBall.svelte    # Main physics simulation component
├── App.svelte                # Main application component
├── app.css                   # Global styles
└── main.ts                   # Application entry point
```

## ⚙️ Physics Parameters Description

| Parameter | Range | Description |
|----------|-------|-------------|
| Gravity Strength | 0.1 - 1.0 | Controls ball falling acceleration |
| Rotation Speed | 0 - 3 | Hexagon rotation speed (degrees/frame) |
| Friction Coefficient | 0.9 - 1.0 | Velocity decay factor |
| Elasticity Coefficient | 0.5 - 1.0 | Energy retention rate after collision |

## 🔧 Core Algorithms

### Collision Detection
Using line-circle intersection algorithm to detect ball collisions with hexagon edges:
```typescript
function pointToLineDistance(point: Point, lineStart: Point, lineEnd: Point): number
```

### Physics Update
```typescript
function updatePhysics() {
  // Apply gravity
  ball.vy += physics.gravity;
  
  // Apply friction
  ball.vx *= physics.friction;
  ball.vy *= physics.friction;
  
  // Update position
  ball.x += ball.vx;
  ball.y += ball.vy;
  
  // Detect collision
  checkCollision();
}
```

## 🎨 Visual Design

- **Dark Tech Theme**: Gradient background with neon colors
- **Glow Effects**: CSS shadows and halos enhance visual hierarchy
- **Responsive Layout**: Self-adapting to different screen sizes
- **Smooth Animation**: requestAnimationFrame ensures smooth experience

## 📱 Responsive Features

- Desktop: Full functionality display
- Tablet: Optimized touch experience
- Mobile: Scale adaptation for small screens

## 🔍 Performance Optimization

- **Canvas Rendering**: Avoid DOM operations, improve rendering performance
- **requestAnimationFrame**: Synchronized with browser refresh rate
- **TypeScript**: Compile-time type checking, reduce runtime errors
- **Component Design**: Modular code structure, easy to maintain

## 🤝 Contribution Guidelines

Welcome to submit Issues and Pull Requests to improve this project!

## 📄 License

MIT License - See [LICENSE](LICENSE) file for details

---

**Enjoy the fun of physics simulation!** 🎯✨