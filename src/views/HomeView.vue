<script setup lang="ts">
import {  doc, onSnapshot, updateDoc } from 'firebase/firestore'
import { onMounted, ref } from 'vue'
import { db } from '@/firebase/index'

const gameBoard = ref<any>(null)
const gameWidth = ref(window.innerWidth)
const gameHeight = ref(window.innerHeight)
const unitSize = ref(20)
const foodX = ref(unitSize.value)
const foodY = ref(0)
const running = ref(false)
const boardBackgroundColor = ref('white')
const snake = ref([{ x: -240, y: -240 }])

const otherSnake = ref([
  { x: unitSize.value * 5, y: 180 },
  { x: unitSize.value * 4, y: 180 },
  { x: unitSize.value * 3, y: 180 },
  { x: unitSize.value * 2, y: 180 },
  { x: unitSize, y: 180 }
])
const score = ref(0)
let xVelocity = unitSize.value
let yVelocity = 0

function resizeCanvas() {
  gameWidth.value = window.innerWidth
  gameHeight.value = window.innerHeight
}

window.addEventListener('resize', resizeCanvas)
window.addEventListener('keydown', (event) => {
  const goingUp = yVelocity == -unitSize.value
  const goingDown = yVelocity == unitSize.value
  const goingRight = xVelocity == unitSize.value
  const goingLeft = xVelocity == -unitSize.value
  const userDoc = doc(db, 'users', 'gC6N1GoEfvH1p3znvozD')

  switch (true) {
    case event.key == 'ArrowLeft' && !goingRight:
      xVelocity = -unitSize.value
      yVelocity = 0
      break
    case event.key == 'ArrowUp' && !goingDown:
      xVelocity = 0
      yVelocity = -unitSize.value
      break
    case event.key == 'ArrowRight' && !goingLeft:
      xVelocity = unitSize.value
      yVelocity = 0
      break
    case event.key == 'ArrowDown' && !goingUp:
      xVelocity = 0
      yVelocity = unitSize.value
      break
  }

  const handleUpdate = async () => {
    await updateDoc(userDoc, {
      snake: snake.value,
      yVelocity: yVelocity,
      xVelocity: xVelocity,
      unitSize: unitSize.value
    })
  }

  handleUpdate()
})

window.addEventListener('touchmove', (e) => {
  console.log(e.touches[0].clientX, e.touches[0].clientY, 'touchmove')
  const posX = Math.round(e.touches[0].clientX / unitSize.value) * unitSize.value
  const posY = Math.round(e.touches[0].clientY / unitSize.value) * unitSize.value
  const snakeHead = { x: snake.value[0].x + xVelocity, y: snake.value[0].y + yVelocity }
  console.log(posX, posY, 'posX, posY', unitSize.value, 'unitSize.value', snakeHead, 'snakeHead')

  switch (true) {
    case posX < snakeHead.x:
      xVelocity = -unitSize.value
      yVelocity = 0
      break
    case posY < snakeHead.y:
      xVelocity = 0
      yVelocity = -unitSize.value
      break
    case posX > snakeHead.x:
      xVelocity = unitSize.value
      yVelocity = 0
      break
    case posY > snakeHead.y:
      xVelocity = 0
      yVelocity = unitSize.value
      break
  }
})

const getUser = async () => {
  const userDoc = doc(db, 'users', 'gC6N1GoEfvH1p3znvozD')
  onSnapshot(userDoc, (userSnapshot) => {
    console.log('yyaaaa');
    
    snake.value = userSnapshot.data()?.snake
    yVelocity = userSnapshot.data()?.yVelocity
    xVelocity = userSnapshot.data()?.xVelocity
    unitSize.value = userSnapshot.data()?.unitSize
  })
}

onMounted(async () => {
  await getUser()
})

onMounted(() => {
  resizeCanvas()

  function startGame() {
    running.value = true
    createFood()
    drawFood()
    nextTick()
  }

  function nextTick() {
    if (running.value) {
      setTimeout(() => {
        comeOutOfOtherSide()
        clearBoard()
        drawFood()
        moveSnake()
        drawSnake()
        drawOtherSnake()
        checkGameOver()
        nextTick()
      }, 100)
    } else {
      displayGameOver()
    }
  }

  function clearBoard() {
    const canvas: any = gameBoard.value
    const ctx: any = canvas.getContext('2d')
    ctx.fillStyle = boardBackgroundColor.value
    ctx.clearRect(0, 0, gameWidth.value, gameHeight.value)
  }

  function createFood() {
    function randomFood(min: number, max: number) {
      const randNum =
        Math.round((Math.random() * (max - min) + min) / unitSize.value) * unitSize.value
      return randNum
    }
    foodX.value = randomFood(0, gameWidth.value - unitSize.value)
    foodY.value = randomFood(0, gameHeight.value - unitSize.value)

    // console.log(foodX.value, foodY.value, 'where food')
  }

  function drawFood() {
    const canvas: any = gameBoard.value

    // Get the 2D rendering context
    const ctx: any = canvas.getContext('2d')
    ctx.fillStyle = 'red'
    ctx.fillRect(foodX.value, foodY.value, unitSize.value, unitSize.value)
  }

  function moveSnake() {
    const snakeHead = { x: snake.value[0].x + xVelocity, y: snake.value[0].y + yVelocity }
    snake.value.unshift(snakeHead)
    // console.log(snakeHead, 'snakeHead')

    if (snakeHead.x == foodX.value && snakeHead.y == foodY.value) {
      score.value += 1
      createFood()
    } else {
      snake.value.pop()
    }
  }

  function drawSnake() {
    const canvas: any = gameBoard.value
    const ctx: any = canvas.getContext('2d')
    snake.value.forEach((snakePart: any) => {
      if (snakePart == snake.value[0]) {
        ctx.fillStyle = 'green'
      } else {
        ctx.fillStyle = 'lightgreen'
        ctx.strokeStyle = 'darkgreen'
      }
      ctx.fillRect(snakePart.x, snakePart.y, unitSize.value, unitSize.value)
      ctx.strokeRect(snakePart.x, snakePart.y, unitSize.value, unitSize.value)
    })
  }

  function drawOtherSnake() {
    const canvas: any = gameBoard.value
    const ctx: any = canvas.getContext('2d')
    otherSnake.value.forEach((snakePart: any) => {
      if (snakePart == otherSnake.value[0]) {
        ctx.fillStyle = 'royalblue'
      } else {
        ctx.fillStyle = 'orange'
        // ctx.strokeStyle = 'red'
      }
      ctx.fillRect(snakePart.x, snakePart.y, unitSize.value, unitSize.value)
      ctx.strokeRect(snakePart.x, snakePart.y, unitSize.value, unitSize.value)
    })
  }

  function changeDirection() {}

  function checkGameOver() {}

  function displayGameOver() {}

  function comeOutOfOtherSide() {
    switch (true) {
      case snake.value[0].x < 0:
        snake.value[0].x = Math.floor(gameWidth.value / unitSize.value) * unitSize.value
        break
      case snake.value[0].x >= gameWidth.value:
        snake.value[0].x = 0
        break
      case snake.value[0].y < 0:
        snake.value[0].y = Math.floor(gameHeight.value / unitSize.value) * unitSize.value
        break
      case snake.value[0].y >= gameHeight.value:
        snake.value[0].y = 0
        break
    }
  }
  startGame()
})
</script>

<template>
  <main>
    <canvas id="gameBoard" ref="gameBoard" :height="gameHeight" :width="gameWidth">
      <button>Hello</button></canvas
    >
  </main>
</template>
