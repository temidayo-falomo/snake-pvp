<script setup lang="ts">
import { onMounted, ref } from 'vue'

const gameBoard = ref<any>(null)
const gameWidth = ref(window.innerWidth)
const gameHeight = ref(window.innerHeight)
const unitSize = ref(25)
const foodX = ref(unitSize.value)
const foodY = ref(0)

function resizeCanvas() {
  gameWidth.value = window.innerWidth
  gameHeight.value = window.innerHeight
  console.log(gameWidth.value, gameHeight.value, 'width and height')
}

window.addEventListener('resize', resizeCanvas)

onMounted(() => {
  resizeCanvas()

  function startGame() {}

  function nextTick() {}

  function clearBoard() {}

  function createFood() {
    function randomFood(min: number, max: number) {
      const randNum =
        Math.round((Math.random() * (max - min) + min) / unitSize.value) * unitSize.value
      return randNum
    }
    foodX.value = randomFood(0, gameWidth.value - unitSize.value)
    foodY.value = randomFood(0, gameHeight.value - unitSize.value)

    console.log(foodX.value, foodY.value, 'where food')
  }

  function drawFood() {
    const canvas: any = gameBoard.value

    // Get the 2D rendering context
    const ctx: any = canvas.getContext('2d')
    ctx.fillStyle = 'red'
    ctx.fillRect(foodX.value, foodY.value, unitSize.value, unitSize.value)
  }

  function moveSnake() {}

  function drawSnake() {}

  function changeDirection() {}

  function checkGameOver() {}

  function displayGameOver() {}

  function resetGame() {}

  createFood()
  drawFood()
})
</script>

<template>
  <main>
    <canvas id="gameBoard" ref="gameBoard" :height="gameHeight" :width="gameWidth"></canvas>
  </main>
</template>
