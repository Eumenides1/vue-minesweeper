<script setup lang="ts" generic="T extends any, O extends any">
interface BlockState {
  x: number
  y: number
  revealed?: boolean
  mine?: boolean
  flagged?: boolean
  adjacentMines: number
}

const WIDTH = 10
const HEIGHT = 10
const state = ref(Array.from({ length: HEIGHT }, (_, y) =>
  Array.from({ length: WIDTH },
    (_, x): BlockState => ({
      x, y, adjacentMines: 0, revealed: false,
    }),
  ),
),
)
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
// 生成炸弹
function generateMines(initial: BlockState) {
  for (const row of state.value) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) < 1)
        continue
      if (Math.abs(initial.y - block.y) < 1)
        continue
      block.mine = Math.random() < 0.3
    }
  }
  updateNumbers()
}

function updateNumbers() {
  state.value.forEach((raw, y) => {
    raw.forEach((block, x) => {
      if (block.mine)
        return
      getSiblings(block).forEach((b) => {
        if (b.mine)
          block.adjacentMines += 1
      })
    })
  })
}

function expendZero(block: BlockState) {
  if (block.adjacentMines)
    return

  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true
      expendZero(s)
    }
  })
}

let mineGenerated = false
const dev = false

function onClick(e: MouseEvent, block: BlockState) {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }
  block.revealed = true
  if (block.mine)
    alert('BOOOOOM!')

  expendZero(block)
}

function onRightClick(block: BlockState) {
  if (block.revealed)
    return
  block.flagged = !block.flagged
}

function getBlockClass(block: BlockState) {
  if (block.flagged)
    return 'bg-gray-500/10'

  if (!block.revealed)
    return 'bg-gray-500/10 hover:bg-gray/10'

  return block.mine ? 'bg-red-500/50' : numberColors[block.adjacentMines]
}

function getSiblings(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx
    const y2 = block.y + dy
    if (x2 < 0 || x2 > WIDTH - 1 || y2 < 0 || y2 > HEIGHT - 1)
      return undefined

    return state.value[y2][x2]
  })
    .filter(Boolean) as BlockState[]
}
watchEffect(checkGameState)

function checkGameState() {
  if (!mineGenerated)
    return
  const blocks = state.value.flat()
  if (blocks.every(block => block.revealed || block.flagged)) {
    if (blocks.some(block => block.flagged && !block.mine))
      alert('You Cheat!!')
    else
      alert('You Win!')
  }
}
</script>

<template>
  <div>
    Minesweeper
    <div p5>
      <div v-for="row, y in state" :key="y" flex="~" items-center justify-center>
        <button
          v-for="block, x in row"
          :key="x"
          :class="getBlockClass(block)"
          flex="~"
          h-10 w-10 m="0.5" items-center justify-center border="1 gray-400/30"
          @click="onClick($event, block)"
          @contextmenu.prevent="onRightClick(block)"
        >
          <template v-if="block.flagged">
            <div i-mdi:flag text-red />
          </template>
          <template v-else-if="block.revealed || dev">
            <div v-if="block.mine" i-mdi:mine />
            <div v-else>
              {{ block.adjacentMines }}
            </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>
