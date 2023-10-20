<template>
  <h2>Gomoku</h2>
  <div>
    <h4>胜负： {{ result }}</h4>
    <div id="pixiRef"></div>
    <div class="controls">
      <button @click="undo">悔棋</button>
      <button @click="restart">重新开始</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import * as PIXI from 'pixi.js';

const pixiRef = ref('');
const undoRef = ref('');
const restartRef = ref('');
const result = ref('-');

let boardSize = 10;
let boardData = Array.from({ length: boardSize }, () =>
  Array.from({ length: boardSize }, () => 0)
);
let currentPlayer = 1;
let history = [];
let isGameOver = false;

const app = new PIXI.Application({
  width: boardSize * 40 + 40,
  height: boardSize * 40 + 40,
  backgroundColor: 0xfeafea,
});

onMounted(() => {
  document.getElementById('pixiRef').appendChild(app.view);
  // pixiRef.value.appendChild(app.view);
});

const board = new PIXI.Graphics();
board.lineStyle(2, 0x000000);
for (let i = 0; i <= boardSize; i++) {
  board.moveTo(20, 20 + i * 40);
  board.lineTo(boardSize * 40 + 20, 20 + i * 40);
  board.moveTo(20 + i * 40, 20);
  board.lineTo(20 + i * 40, boardSize * 40 + 20);
}
app.stage.addChild(board);

const chesses = [];

app.view.addEventListener('click', (event) => {
  if (isGameOver) {
    return;
  }
  const x = event.offsetX;
  const y = event.offsetY;
  const i = Math.round((x - 20) / 40);
  const j = Math.round((y - 20) / 40);
  if (i < 0 || i >= boardSize || j < 0 || j >= boardSize) {
    return;
  }
  if (boardData[i][j] !== 0) {
    return;
  }
  boardData[i][j] = currentPlayer;
  history.push([i, j]);
  const newChess = new PIXI.Graphics();
  newChess.beginFill(currentPlayer === 1 ? 0xffffff : 0x000000);
  newChess.drawCircle(0, 0, 18);
  newChess.endFill();
  newChess.position.set(20 + i * 40, 20 + j * 40);
  app.stage.addChild(newChess);
  chesses.push(newChess);
  if (checkWin(i, j)) {
    setTimeout(() => {
      result.value = `${currentPlayer === 1 ? '白' : '黑'}棋胜利！`;
    }, 100);
    isGameOver = true;
    return;
  }
  currentPlayer = -currentPlayer;
});

function checkWin(i, j) {
  const directions = [
    [1, 0],
    [0, 1],
    [1, 1],
    [1, -1],
  ];
  for (const [dx, dy] of directions) {
    let count = 1;
    for (let k = 1; k < 5; k++) {
      const x = i + k * dx;
      const y = j + k * dy;
      if (
        x < 0 ||
        x >= boardSize ||
        y < 0 ||
        y >= boardSize ||
        boardData[x][y] !== currentPlayer
      ) {
        break;
      }
      count++;
    }
    for (let k = 1; k < 5; k++) {
      const x = i - k * dx;
      const y = j - k * dy;
      if (
        x < 0 ||
        x >= boardSize ||
        y < 0 ||
        y >= boardSize ||
        boardData[x][y] !== currentPlayer
      ) {
        break;
      }
      count++;
    }
    if (count >= 5) {
      return true;
    }
  }
  return false;
}

const undo = () => {
  if (isGameOver || history.length === 0) {
    return;
  }
  const [i, j] = history.pop();
  boardData[i][j] = 0;
  currentPlayer = -currentPlayer;
  const chess = chesses.pop();
  app.stage.removeChild(chess);
};

const restart = () => {
  boardData = Array.from({ length: boardSize }, () =>
    Array.from({ length: boardSize }, () => 0)
  );
  currentPlayer = 1;
  history = [];
  isGameOver = false;
  app.stage.removeChildren();
  board.clear();
  board.lineStyle(2, 0x000000);
  for (let i = 0; i <= boardSize; i++) {
    board.moveTo(20, 20 + i * 40);
    board.lineTo(boardSize * 40 + 20, 20 + i * 40);
    board.moveTo(20 + i * 40, 20);
    board.lineTo(20 + i * 40, boardSize * 40 + 20);
  }
  app.stage.addChild(board);
  chesses.length = 0;
  result.value = `-`;
};
</script>
