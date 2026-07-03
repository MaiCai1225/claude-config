# 桌面番茄钟 — 实施方案

## 背景

用户需要一款桌面番茄钟软件。采用**单 HTML 文件**方案（内嵌 CSS/JS），零依赖，双击即用，在任何浏览器中运行。界面精美现代，功能完整。

## 最终产物

一个文件：`C:\Users\Blackmai\Desktop\pomodoro.html`

## 功能清单

### 核心功能
- ⏱ **工作/休息模式**：25 分钟工作 → 5 分钟短休息 → 每 4 轮后一次 15 分钟长休息
- 🔴🟢🔵 **三色调**：工作=番茄红，短休息=绿，长休息=蓝
- ⭕ **圆形进度环**：SVG 环形倒计时，平滑动画
- ▶️⏸️🔄 **开始/暂停/重置** 三个按钮
- 🔢 **会话计数**：显示当前轮次和完成的番茄数
- 🔔 **音效提醒**：Web Audio API 生成提示音（无需外部音频文件）
- 🔔 **桌面通知**：Notification API 弹出系统通知

### 顺带功能
- ⚙️ **设置面板**：可自定义工作时长、短休息、长休息、长休息间隔（几轮）
- 📋 **任务列表**：添加/删除/完成当前任务
- ⌨️ **键盘快捷键**：空格=开始/暂停，R=重置，S=跳过当前阶段
- 🏷️ **标签页图标倒计时**：在 favicon 位置显示剩余分钟数
- 💾 **LocalStorage 持久化**：保存设置和历史记录

## 架构设计

### 状态机

```
states: idle → running → paused → (timer ends) → break → running → ...
```

用一个 `state` 变量管理：
- `idle` — 初始状态
- `running` — 正在计时
- `paused` — 已暂停
- `break` — 休息时间

加上 `mode` 变量：
- `work` / `shortBreak` / `longBreak`

### 数据结构

```js
const CONFIG = {
  workDuration: 25 * 60,       // 秒
  shortBreakDuration: 5 * 60,
  longBreakDuration: 15 * 60,
  longBreakInterval: 4,        // 每 N 轮后长休息
}

const STATE = {
  mode: 'work',                // 'work' | 'shortBreak' | 'longBreak'
  timerState: 'idle',          // 'idle' | 'running' | 'paused'
  remainingSeconds: 1500,
  completedSessions: 0,        // 本轮已完成番茄数
  totalPomodoros: 0,           // 历史总番茄数
  currentTask: '',
  tasks: [],
}
```

### HTML 结构

```
.container
  .timer-section
    svg.progress-ring  (圆形进度)
    .timer-display      (MM:SS)
    .mode-label         (专注 / 短休息 / 长休息)
  .controls
    button.toggle      (开始/暂停)
    button.reset       (重置)
    button.skip        (跳过)
  .session-info        (第 X 轮 / 已完成 N 个番茄)
  .task-section
    input + add-btn
    ul.task-list
  .settings-panel     (折叠式)
    各时长滑块
```

### 核心技术点

1. **计时器**：`setInterval` 每秒更新，计算进度百分比驱动 SVG 环形进度条
2. **进度环**：SVG `<circle>` 的 `stroke-dasharray` / `stroke-dashoffset` 实现
3. **提示音**：`AudioContext` + `OscillatorNode` 生成蜂鸣声
4. **桌面通知**：`new Notification('番茄钟', { body: '...' })` — 需用户授权
5. **Favicon 倒计时**：用 `<canvas>` 绘制数字 → `link[rel=icon]` 替换
6. **持久化**：`localStorage` 读写配置和历史

### 视觉设计

- 深色背景 (dark mode by default)，柔和圆角
- 大号圆形进度环居中
- 番茄红 `#FF6347` / 绿 `#4CAF50` / 蓝 `#2196F3`
- 毛玻璃效果的控制按钮
- CSS transition 做按钮和模式切换动画
- 响应式布局，桌面端和移动端均可用

## 验证步骤

1. 双击 `pomodoro.html` 在浏览器中打开
2. 点击「开始」— 计时器开始倒计时，进度环转动
3. 等待计时结束 — 听到提示音 + 弹出通知
4. 自动切换到休息模式（颜色变化）
5. 测试暂停/恢复/重置功能
6. 打开设置，修改时长，确认生效
7. 添加任务，确认增删功能正常
8. 刷新页面，确认设置和进度持久化
9. 测试键盘快捷键（空格、R、S）
