<!-- eslint-disable no-alert -->
<!-- eslint-disable no-console -->
<script setup lang="ts">
import type { BlockState } from '~/types'
const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
]
const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-teal-500',
]
defineOptions({
  name: 'IndexPage',
})

const Width = 5
const Height = 5
const state = ref(Array.from({ length: Height },
  (_, y) => Array.from({ length: Width },
    (_, x): BlockState => ({
      x,
      y,
      adjacentMines: 0,
      revealed: false,
    }))))
let mineGenerated = false
const dev = false

function getBlockClass(block: BlockState) {
  if (block.flagged)
    return 'bg-gray-500/10'
  if (!block.revealed)
    return 'bg-gray-500/10 hover: bg-gray/10'
  return block.mine ? 'bg-red-500/50' : numberColors[block.adjacentMines]
}

function generateMines(initial: BlockState) {
  for (const row of state.value) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) < 1)
        continue
      if (Math.abs(initial.y - block.y) < 1)
        continue
      block.mine = Math.random() < 0.2
    }
  }
  updateNumbers()
}

function updateNumbers() {
  state.value.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine)
        return
      getSiblings(block).forEach((b) => {
        if (b.mine)
          block.adjacentMines += 1
      })
    })
  })
}

function getSiblings(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx
    const y2 = block.y + dy
    if (x2 < 0 || x2 >= Width || y2 < 0 || y2 >= Height)
      return undefined
    return state.value[y2][x2]
  }).filter(Boolean) as BlockState[]
}

function onClick(e: MouseEvent, block: BlockState) {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }
  block.revealed = true
  console.log(block)
  if (block.mine)
    alert('BOOM!!!')
  expandZero(block)
}

function onRightClick(block: BlockState) {
  if (block.revealed)
    return
  block.flagged = !block.flagged
}
function expandZero(block: BlockState) {
  if (block.adjacentMines)
    return
  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true
      expandZero(s)
    }
  })
}
watchEffect(checkGameState)
function checkGameState() {
  if (!mineGenerated)
    return
  const blocks = state.value.flat()
  if (blocks.every(block => block.revealed || block.flagged)) {
    if (blocks.some(block => block.flagged && !block.mine))
      alert('you cheat')
    else
      alert('you win')
  }
}
</script>

<template>
  <div>
    Minesweeper
    <div p5>
      <div
        v-for="row, y in state" :key="y" flex="~" justify-center
        items-center
      >
        <button
          v-for="item, x in row"
          :key="x"
          w-10
          h-10
          m="0.5"
          border="1 gray-400/10"
          :class="getBlockClass(item)"
          flex="~"
          justify-center
          items-center
          @click="onClick($event, item)"
          @contextmenu.prevent="onRightClick(item)"
        >
          <template v-if="item.flagged">
            <div i-mdi-flag text-red />
          </template>
          <template v-if="item.revealed || dev">
            <div v-if="item.mine" i-mdi-mine />
            <div v-else>
              {{ item.adjacentMines }}
            </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>

<route lang="yaml">
meta:
  layout: home
</route>
