<script setup lang="ts">
import { onMounted, ref } from 'vue'

// const myCanvas = ref<any>(null)
// const ctx = myCanvas.value.getContext('2d')

const gameBoard = ref(null)

const width = ref(500)
const height = ref(500)

onMounted(() => {
  // Get the canvas element using refs
  const canvas: any = gameBoard.value

  // Get the 2D rendering context
  const ctx: any = canvas.getContext('2d')

  const foodY = ref(0)
  const foodX = ref(0)

  function resizeCanvas() {
    width.value = window.innerWidth
    height.value = window.innerHeight
  }

  // window.addEventListener('resize', resizeCanvas)

  // resizeCanvas()

  const boardBackground = 'white'
  const snakeColor = 'lightgreen'
  const snakeBorder = 'black'
  const foodColor = 'red'
  const unitSize = 25
  const running = ref(false)

  let xVelocity = 25
  let yVelocity = 0

  let score = 0

  let snake = [
    {
      x: unitSize * 4,
      y: 0
    },
    {
      x: unitSize * 3,
      y: 0
    },
    {
      x: unitSize * 2,
      y: 0
    },
    {
      x: unitSize,
      y: 0
    },
    {
      x: 0,
      y: 0
    }
  ]

  window.addEventListener('keydown', (event) => {
    const goingUp = yVelocity == -unitSize
    const goingDown = yVelocity == unitSize
    const goingRight = xVelocity == unitSize
    const goingLeft = xVelocity == -unitSize

    switch (true) {
      case event.key == 'ArrowLeft' && !goingRight:
        xVelocity = -unitSize
        yVelocity = 0
        break
      case event.key == 'ArrowUp' && !goingDown:
        xVelocity = 0
        yVelocity = -unitSize
        break
      case event.key == 'ArrowRight' && !goingLeft:
        xVelocity = unitSize
        yVelocity = 0
        break
      case event.key == 'ArrowDown' && !goingUp:
        xVelocity = 0
        yVelocity = unitSize
        break
    }
  })

  function gameStart() {
    running.value = true
    // scoreText.textContent = score
    createFood()
    drawFood()
    nextTick()
  }

  function nextTick() {
    console.log('nextTick')
    if (running.value) {
      setTimeout(() => {
        clearBoard()
        drawFood()
        moveSnake()
        drawSnake()
        comeOutOfOtherSide()
        checkGameOver()
        nextTick()
      }, 1000)
    } else {
      displayGameOver()
    }
  }

  function clearBoard() {
    ctx.fillStyle = boardBackground
    // ctx.strokeStyle = snakeBorder;
    ctx.fillRect(0, 0, width, height)
  }

  function createFood() {
    function randomFood(min: any, max: any) {
      const randNum = Math.round((Math.random() * (max - min) + min) / unitSize) * unitSize
      return randNum
    }
    foodX.value = randomFood(0, width.value - unitSize)
    foodY.value = randomFood(0, width.value - unitSize)
  }

  function drawFood() {
    ctx.fillStyle = foodColor
    // ctx.strokeStyle = snakeBorder;
    ctx.fillRect(foodX, foodY, unitSize, unitSize)
    ctx.strokeRect(foodX, foodY, unitSize, unitSize)
  }

  function moveSnake() {
    const head = {
      x: snake[0].x + xVelocity,
      y: snake[0].y + yVelocity
    }
    snake.unshift(head)
    //if food is eaten
    if (snake[0].x == foodX.value && snake[0].y == foodY.value) {
      score += 1
      // scoreText.textContent = score
      createFood()
    } else {
      snake.pop()
    }

    console.log(snake)
  }

  function drawSnake() {
    ctx.fillStyle = snakeColor
    ctx.strokeStyle = snakeBorder
    snake.forEach((unit) => {
      ctx.fillRect(unit.x, unit.y, unitSize, unitSize)
      ctx.strokeRect(unit.x, unit.y, unitSize, unitSize)
    })
  }

  function checkGameOver() {
    for (let i = 1; i < snake.length; i += 1) {
      if (snake[i].x == snake[0].x && snake[i].y == snake[0].y) {
        running.value = false
      }
    }
  }

  function comeOutOfOtherSide() {
    switch (true) {
      case snake[0].x < 0:
        snake[0].x = width.value
        break
      case snake[0].x >= width.value:
        snake[0].x = 0
        break
      case snake[0].y < 0:
        snake[0].y = height.value
        break
      case snake[0].y >= height.value:
        snake[0].y = 0
        break
    }
  }

  function displayGameOver() {
    ctx.font = '50px MV Boli'
    ctx.fillStyle = 'black'
    ctx.textAlign = 'center'
    ctx.fillText('GAME OVER!', width.value / 2, height.value / 2)
    running.value = false
  }

  function resetGame() {
    score = 0
    xVelocity = unitSize
    yVelocity = 0
    snake = [
      { x: unitSize * 4, y: 0 },
      { x: unitSize * 3, y: 0 },
      { x: unitSize * 2, y: 0 },
      { x: unitSize, y: 0 },
      { x: 0, y: 0 }
    ]
    running.value = false
    checkGameOver()
    gameStart()
  }

  gameStart()
})
</script>

<template>
  <main>
    <canvas
      id="gameBoard"
      ref="gameBoard"
      :style="{ width: width + 'px', height: height + 'px' }"
    ></canvas>
  </main>
</template>
