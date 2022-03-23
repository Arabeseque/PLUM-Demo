<script setup lang="ts">

const el = $ref<HTMLCanvasElement>()
const ctx = $computed(() => el!.getContext('2d')!)
const WIDTH = 600
const HEIGHT = 600

interface Point {
  x: number
  y: number
}

interface Branch {
  start: Point
  length: number
  theta: number
}

function lineTo(p1: Point, p2: Point) {
  ctx.beginPath()
  ctx.moveTo(p1.x, p1.y)
  ctx.lineTo(p2.x, p2.y)
  ctx.stroke()
}
function getEndPoint(b: Branch) {
  return {
    x: b.start.x + b.length * Math.cos(b.theta),
    y: b.start.y + b.length * Math.sin(b.theta),
  }
}

const pendingTasks: Function[] = []
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

function drawBranch(b: Branch) {
  lineTo(b.start, getEndPoint(b))
}
function init() {
  ctx.strokeStyle = '#fff'
  const branch = {
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 20,
    theta: -Math.PI / 2,
  }
  step(branch)
  // const startPoint = { x: WIDTH / 2, y: HEIGHT }
  // const endPoint = { x: WIDTH / 2, y: HEIGHT / 2 }
  // line(startPoint, endPoint)
}

onMounted(() => {
  init()
})
</script>

<template>
  <canvas ref="el" width="600" height="600" border>
  <!-- <main font-sans p="x-4 y-10" text="center gray-700 dark:gray-200">
    <router-view />
  </main> -->
  </canvas>
  <Footer />
</template>
