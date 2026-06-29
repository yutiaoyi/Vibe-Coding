# Cute Assets Gallery Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 新增一个独立 HTML（p5.js）文件，按网格画册一次性渲染 40 个“温暖可爱、手绘抖动风”的资产（中英双语标签），并支持点击单格随机刷新该资产的细节。

**Architecture:** 复用 `shadow-town.html` 的“手绘风工具函数”（抖动线/多边形、草图描边）与随机策略；每个资产实现为 `drawAsset(ctx)` 风格的独立函数，通过 `randomSeed` 在每格可重复渲染；主 `draw()` 负责背景、网格布局、标签与交互高亮。

**Tech Stack:** p5.js（CDN 引入），纯前端单文件 HTML。

---

## File structure

**Create**
- `cute-assets-gallery.html`: 独立可打开运行的画册页（包含 p5.js + 资产绘制代码）

**Reference only**
- `shadow-town.html`: 复用其中的手绘风函数与风格参数（不修改原文件）

---

### Task 1: Define asset list + grid layout contract

**Files:**
- Create: `cute-assets-gallery.html`

- [ ] **Step 1: Define the 40 assets metadata (CN/EN)**

在 HTML 的脚本中定义：

```js
const ASSETS = [
  { id: "sheep", nameCN: "绵羊", nameEN: "Sheep", draw: drawSheep },
  // ... total 40
];
```

资产 1-20（保留用户已确认的那批）：
1. 绵羊 / Sheep
2. 小狗 / Puppy
3. 小猫 / Kitty
4. 小兔子 / Bunny
5. 小鸭子 / Duckling
6. 小刺猬 / Hedgehog
7. 小熊 / Bear
8. 小狐狸 / Fox
9. 企鹅 / Penguin
10. 小鸟 / Bird
11. 蜗牛 / Snail
12. 毛毛虫 / Caterpillar
13. 小花盆植物 / Potted Plant
14. 蘑菇屋 / Mushroom Hut
15. 礼物盒 / Gift Box
16. 热气球 / Hot Air Balloon
17. 风筝 / Kite
18. 热可可杯 / Cocoa Mug
19. 邮筒 / Postbox
20. 长椅+灯串 / Bench + Fairy Lights

资产 21-40（道具为主，用户选择 B）：
21. 面包店招牌 / Bakery Sign
22. 糖果车 / Candy Cart
23. 小帐篷 / Tent
24. 野餐篮 / Picnic Basket
25. 风铃 / Wind Chime
26. 纸飞机 / Paper Plane
27. 气球束 / Balloons
28. 小路牌 / Direction Sign
29. 小屋式邮箱 / Mailbox
30. 小喷泉 / Fountain
31. 小桥 / Little Bridge
32. 路边花丛 / Flower Bush
33. 热可可摊位 / Cocoa Stall
34. 小书摊 / Book Stand
35. 灯串（单独）/ String Lights
36. 雨伞 / Umbrella
37. 风车玩具 / Pinwheel
38. 贴纸徽章 / Sticker Badge
39. 小钟楼 / Clock Tower
40. 猫爪脚印 / Paw Prints
```

- [ ] **Step 2: Decide grid size and cell contract**

推荐在桌面端优先使用 \(8 \times 5\)：

```js
const GRID = { cols: 8, rows: 5, pad: 18, labelH: 28 };
// cell rect: x,y,w,h where h includes drawing region + label region
```

- [ ] **Step 3: Verification**

打开 `cute-assets-gallery.html`，能看到空网格与 40 个双语标签占位（先用简单 placeholder draw 函数）。

---

### Task 2: Port sketchy drawing utilities (from shadow-town)

**Files:**
- Create/Modify: `cute-assets-gallery.html`

- [ ] **Step 1: Copy minimal utilities**

从 `shadow-town.html` 迁移并保留最小闭包：
- `subdivideLine`
- `subdividePolygon`
- `drawSketchyPoly`
- `drawSketchyLineSegments`
- `lerpArr`

并用一个统一的风格配置：

```js
const STYLE = {
  stroke: [60, 40, 30],
  strokeW: 2.2,
  jitter: 1.2,
  polySeg: 3,
  bg: [248, 238, 220],
};
```

- [ ] **Step 2: Verification**

刷新页面，线条抖动可见、整体观感接近 `shadow-town.html`（非像素完美，但同类风格）。

---

### Task 3: Implement 40 asset draw functions

**Files:**
- Modify: `cute-assets-gallery.html`

- [ ] **Step 1: Standardize asset API**

每个资产函数统一签名：

```js
function drawSheep({ x, y, s, seed, hovered }) {
  push();
  randomSeed(seed);
  translate(x, y);
  // draw within a canonical box around (0,0)
  pop();
}
```

- [ ] **Step 2: Implement assets using primitives only**

约束（保证风格一致与工作量可控）：
- 只用线段/多边形/圆/弧（p5 的 `line/ellipse/arc/beginShape`）
- 边缘必须走 `subdivideLine/subdividePolygon` 或画两遍轻微错位，形成手绘感
- 颜色使用“暖色系低饱和”，夜景/发光不做（这是画册页，不是场景）

每个资产至少包含：
- 主体轮廓（poly/ellipse）
- 1-3 个小细节（眼睛/腮红/图案/挂件）
- 可选：`hovered` 时加一个轻微高亮（例如描边加粗或淡光晕）

- [ ] **Step 3: Verification**

打开页面能看到 40 个资产，不重叠、可辨识、风格统一。

---

### Task 4: Grid renderer + bilingual labels + interaction

**Files:**
- Modify: `cute-assets-gallery.html`

- [ ] **Step 1: Render grid + labels**

每格绘制顺序：
- 背景卡片（圆角 rect）
- 资产（居中缩放到绘制区域）
- 标签（两行：CN 上、EN 下）

- [ ] **Step 2: Add hover + click-to-reroll (per cell)**

实现：
- 预生成 `cellSeeds[i]`
- `mouseMoved` 计算 hover index
- `mousePressed`：如果点击在某个 cell 内，刷新该 cell 的 seed（例如 `cellSeeds[i] = floor(random(1e9))`）

- [ ] **Step 3: Verification**

鼠标移到单格能看到高亮；点击单格，该资产的“随机微细节”（比如表情、条纹、摆件角度）会改变。

---

### Task 5: Polish + responsiveness

**Files:**
- Modify: `cute-assets-gallery.html`

- [ ] **Step 1: Responsive grid**

窗口过窄时自动降列数（例如 8→6→4），保持 40 个可滚动或自动换行；或保持固定行列但缩小单格。

- [ ] **Step 2: Add small on-canvas help text**

右下角加一句：
- “Hover 高亮，Click 重掷单格细节”

- [ ] **Step 3: Verification**

缩放窗口后布局依然可用，文字不遮挡资产。

---

## How to run / test

- 直接用浏览器打开：`cute-assets-gallery.html`
- 预期：出现 40 格画册，每格有双语标签；悬停高亮；点击该格资产细节变化。

