# PLUM demo

使用模板

* vitesse lite (vue3实验性功能)
* uno css

```
npx degit antfu/vitesse-lite my-vitesse-app
pnpm i
```

## 思路

canvas画线

```javascript
interface Point {
  x: number
  y: number
}
function lineTo(p1: Point, p2: Point) {
  ctx.beginPath()
  ctx.moveTo(p1.x, p1.y)
  ctx.lineTo(p2.x, p2.y)
  ctx.stroke()
}
```

给定Point,画出线段

```javascript
interface Branch {
  start: Point
  length: number
  theta: number
}
function getEndPoint(b: Branch) {
  return {
    x: b.start.x + b.length * Math.cos(b.theta),
    y: b.start.y + b.length * Math.sin(b.theta),
  }
}
function drawBranch(b: Branch) {
  lineTo(b.start, getEndPoint(b))
}
```

通过递归实现自由划线

```javascript
function step(b: Branch, depth = 0) {
  const end = getEndPoint(b)
  drawBranch(b)
  if (depth < 3 || Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: end,
      length: b.length + (Math.random() * 10 - 5),
      theta: b.theta - 0.3 * Math.random(),
    }, depth + 1))
  }
  if (depth < 3 || Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: end,
      length: b.length + (Math.random() * 10 - 5),
      theta: b.theta + 0.3 * Math.random(),
    }, depth + 1))
  }
}
```

需求：

* 实现随帧变化的画面



JavaScript pending实现广度优先

* 生成完一根树枝之后同时生成一根树枝的左右

使用 pendingTasks

```javascript
const pendingTasks: Function[] = []
function step(b: Branch) {
 	...
}
function frame() {
  const tasks = [...pendingTasks]
  pendingTasks.length = 0
  tasks.forEach(fn => fn())
}

let framesCount = 0
function startFrame() {
  requestAnimationFrame(() => {
    framesCount += 1
    if (framesCount % 3 === 0)
      frame()
    startFrame()
  })
}
startFrame()
```

requestiAnimationFrame 和 setTimeInteval的差别

* requestiAnimationFrame()按照浏览器帧变化进行画图，减少卡顿的的产生

调用函数

```javascript
function init() {
  ctx.strokeStyle = '#fff'
  const branch = {
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 20,
    theta: -Math.PI / 2,
  }
  step(branch)
}
onMounted(() => {
  init()
})
```

