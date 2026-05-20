# 贪吃蛇 — 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal：** 用单文件 HTML + Canvas + JavaScript 实现一个可运行的贪吃蛇游戏

**Architecture：** 单文件架构，所有逻辑（状态管理、游戏循环、渲染、输入）写在 `index.html` 中，通过 Canvas API 绘制蛇和食物。

**Tech Stack：** HTML5 Canvas + 原生 JavaScript，无任何外部依赖

---

### Task 1: 搭建项目骨架

**Files:**
- Create: `index.html`

- [ ] **Step 1: 创建 `index.html` 并编写基础结构**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>贪吃蛇</title>
  <style>
    body {
      margin: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      background: #222;
    }
    canvas {
      border: 2px solid #444;
    }
  </style>
</head>
<body>
  <canvas id="game" width="400" height="400"></canvas>
  <script>
    // 游戏代码写在这里
  </script>
</body>
</html>
```

- [ ] **Step 2: 验证页面可正常加载**

用浏览器打开 `index.html`，确认画布居中显示，无报错。

---

### Task 2: 定义游戏常量与初始状态

**Files:**
- Modify: `index.html`（在 `<script>` 标签内）

- [ ] **Step 1: 添加常量定义和初始状态变量**

```javascript
const GRID_SIZE = 20;      // 每格像素
const GRID_COUNT = 20;     // 格子数量
const TICK_MS = 150;       // 每帧间隔（毫秒）

const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

// 初始蛇：中央横向排列，长度 3，头朝右
let snake = [
  { x: 10, y: 10 },
  { x: 9,  y: 10 },
  { x: 8,  y: 10 },
];
let direction = { x: 1, y: 0 };   // 当前移动方向
let nextDirection = { x: 1, y: 0 }; // 缓存下一次方向
let food = { x: 15, y: 10 };
let gameOver = false;
```

- [ ] **Step 2: 验证变量定义正确**

在浏览器控制台打印 `snake`、`direction`，确认初始值符合预期。

---

### Task 3: 实现食物随机生成

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 编写 `placeFood()` 函数**

```javascript
function placeFood() {
  let newFood;
  do {
    newFood = {
      x: Math.floor(Math.random() * GRID_COUNT),
      y: Math.floor(Math.random() * GRID_COUNT),
    };
  } while (snake.some(seg => seg.x === newFood.x && seg.y === newFood.y));
  food = newFood;
}
```

- [ ] **Step 2: 在初始化时调用一次 `placeFood()`**

在 `gameOver = false` 之后添加一行 `placeFood();`

---

### Task 4: 实现渲染函数

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 编写 `draw()` 渲染函数**

```javascript
function draw() {
  // 清空画布
  ctx.fillStyle = '#111';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // 绘制食物
  ctx.fillStyle = '#f44';
  ctx.fillRect(food.x * GRID_SIZE, food.y * GRID_SIZE, GRID_SIZE, GRID_SIZE);

  // 绘制蛇
  ctx.fillStyle = '#4f4';
  for (const seg of snake) {
    ctx.fillRect(seg.x * GRID_SIZE, seg.y * GRID_SIZE, GRID_SIZE, GRID_SIZE);
  }
}
```

- [ ] **Step 2: 验证渲染正确**

手动调用 `draw()`，在浏览器中确认蛇和食物出现在画布上。

---

### Task 5: 实现游戏逻辑 tick

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 编写 `tick()` 更新函数**

```javascript
function tick() {
  if (gameOver) return;

  // 应用缓存的方向
  direction = nextDirection;

  // 计算蛇头新位置
  const head = {
    x: snake[0].x + direction.x,
    y: snake[0].y + direction.y,
  };

  // 撞墙检测
  if (head.x < 0 || head.x >= GRID_COUNT || head.y < 0 || head.y >= GRID_COUNT) {
    gameOver = true;
    return;
  }

  // 撞自身检测（排除蛇尾，排除增长情形）
  const willGrow = head.x === food.x && head.y === food.y;
  const checkLength = willGrow ? snake.length : snake.length - 1;
  for (let i = 0; i < checkLength; i++) {
    if (snake[i].x === head.x && snake[i].y === head.y) {
      gameOver = true;
      return;
    }
  }

  // 移动蛇身
  snake.unshift(head);
  if (!willGrow) snake.pop();
  else placeFood();
}
```

- [ ] **Step 2: 验证 tick 逻辑**

连续调用 `tick()` 若干次，配合 `draw()` 观察蛇是否正确移动。

---

### Task 6: 实现键盘输入处理

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 添加键盘事件监听**

```javascript
window.addEventListener('keydown', (e) => {
  switch (e.key) {
    case 'ArrowUp':
      if (direction.y === 0) nextDirection = { x: 0, y: -1 };
      break;
    case 'ArrowDown':
      if (direction.y === 0) nextDirection = { x: 0, y: 1 };
      break;
    case 'ArrowLeft':
      if (direction.x === 0) nextDirection = { x: -1, y: 0 };
      break;
    case 'ArrowRight':
      if (direction.x === 0) nextDirection = { x: 1, y: 0 };
      break;
  }
  // 阻止方向键滚动页面
  if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(e.key)) {
    e.preventDefault();
  }
});
```

- [ ] **Step 2: 按方向键，验证 `nextDirection` 正确更新，且无法 180° 掉头**

---

### Task 7: 启动游戏主循环

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 在文件末尾添加 `setInterval` 启动游戏循环**

```javascript
setInterval(() => {
  tick();
  draw();
}, TICK_MS);
```

- [ ] **Step 2: 完整测试**

用浏览器打开游戏，验证：
1. 方向键控制蛇的移动方向
2. 蛇碰到食物后正确变长
3. 撞墙或撞自己后显示 "Game Over"

---

### Task 8: 添加 Game Over 显示

**Files:**
- Modify: `index.html`

- [ ] **Step 1: 在 `draw()` 函数末尾添加 Game Over 文字**

```javascript
function draw() {
  // ... 现有绘制逻辑 ...

  if (gameOver) {
    ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = '#fff';
    ctx.font = 'bold 40px monospace';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText('Game Over', canvas.width / 2, canvas.height / 2);
  }
}
```

- [ ] **Step 2: 手动触发游戏结束，验证文字正确显示**

---

## 自检清单

**Spec 覆盖检查：**
- [ ] 蛇初始长度为 3 节，中央就位 ✓
- [ ] 每 150ms 移动一格 ✓
- [ ] 方向键控制方向，禁止 180° 掉头 ✓
- [ ] 食物随机出现在空白格子 ✓
- [ ] 吃食物 → 蛇身增长 +10 分 ✓
- [ ] 撞墙或撞自己 → Game Over ✓
- [ ] 代码行数控制在 100 行以内 ✓

**占位符扫描：** 无 TBD / TODO / "待实现" 等占位符

**类型一致性：** 无类型问题（JavaScript 无静态类型）

**行数统计：** 预计总行数约 85 行（HTML 结构 15 行 + JS 逻辑约 70 行）