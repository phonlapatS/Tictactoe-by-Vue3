<script setup>
import { ref, computed } from "vue";

const player = ref("X");
const board = ref([
  ["", "", ""],
  ["", "", ""],
  ["", "", ""],
]);

// หาผู้ชนะ
const CalculateWinner = (squares) => {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
};

const winner = computed(() => CalculateWinner(board.value.flat()));

// ตรวจสอบว่า "เสมอ" หรือไม่
const isDraw = computed(() => {
  return board.value.flat().every(cell => cell !== '') && !winner.value;
});

// ให้ Player เดิน
const MakeMove = (x, y) => {
  if (winner.value || isDraw.value || board.value[x][y] !== '') return;

  board.value[x][y] = player.value;
  player.value = player.value === 'X' ? 'O' : 'X';

  if (!winner.value && !isDraw.value && player.value === 'O') {
    MakeAIMove();
  }
};

// ฟังก์ชั่นให้ AI เดิน
const MakeAIMove = () => {
  // เพื่อหาการเดินที่ดีที่สุดของ AI
  const findBestMove = (mark) => {
    // ลูปวนตรวจสอบทุกช่องในกระดาน
    for (let x = 0; x < 3; x++) {
      for (let y = 0; y < 3; y++) {
        // ตรวจสอบว่ามีช่องว่างอยู่หรือไม่
        if (board.value[x][y] === '') {
          // จำลองการลงหมากในช่องว่าง
          board.value[x][y] = mark;
          // ตรวจสอบว่าการเดินช่องนี้ทำให้ชนะหรือไม่
          if (CalculateWinner(board.value.flat()) === mark) {
            // รีเซ็ตกระดานก่อนส่งคืนตำแหน่งที่ดีที่สุด
            board.value[x][y] = '';
            return { x, y }; // ส่งตำแหน่งการเดินที่ดีที่สุด
          }
          // รีเซ็ตช่องนั้นให้ว่างถ้าการเดินนั้นไม่ทำให้ชนะ
          board.value[x][y] = '';
        }
      }
    }
    return null; // ถ้าไม่มีการเดินที่ทำให้ชนะ คืนค่าเป็น null
  };

  // ลองหาตำแหน่งที่ทำให้ AI ชนะ
  let move = findBestMove("O");

  // ถ้าไม่มีตำแหน่งที่ทำให้ AI ชนะ ให้บล็อกการเดินของ player ที่อาจจะชนะ
  if (!move) {
    move = findBestMove("X");
  }

  // ถ้าไม่มีการเดินที่ทำให้ชนะหรือบล็อกได้ ให้เลือกช่องตรงกลางถ้าว่างอยู่
  if (!move) {
    if (board.value[1][1] === '') {
      move = { x: 1, y: 1 };
    }
  }

  // ถ้าช่องตรงกลางไม่ว่าง ให้เลือกช่องอื่นๆ
  if (!move) {
    for (let x = 0; x < 3; x++) {
      for (let y = 0; y < 3; y++) {
        if (board.value[x][y] === '') {
          move = { x, y };
          break; // เจอช่องว่างแล้วออกจากลูป
        }
      }
      if (move) break;
    }
  }

  // ถ้าหาตำแหน่งได้แล้ว ให้ AI เดินและสลับเป็นตาของ player ต่อ
  if (move) {
    board.value[move.x][move.y] = 'O';
    player.value = 'X';
  }
};

// รีเซ็ตเกม
const ResetGame = () => {
  board.value = [
    ["", "", ""],
    ["", "", ""],
    ["", "", ""],
  ];

  // สุ่มว่าฝั่งไหนจะเริ่มก่อน X หรือ O
  player.value = Math.random() < 0.5 ? "X" : "O";

  // ถ้า AI ถูกเลือกให้เริ่มก่อน ให้ AI เดินทันที
  if (player.value === "O") {
    MakeAIMove();
  }
};
</script>

<template>
  <main class="pt-8 text-center dark:bg-gray-800 min-h-screen dark:text-white">
    <h1 class="mb-8 text-3xl font-bold uppercase">Tic Tac Toe</h1>

    <h3 v-if="!winner && !isDraw" class="text-xl mb-4">ถึงตาผู้เล่น {{ player }}</h3>

    <div class="flex flex-col items-center mb-8">
      <div v-for="(row, x) in board" :key="x" class="flex">
        <div
          v-for="(cell, y) in row"
          :key="y"
          @click="MakeMove(x, y)"
          :class="`border border-black w-20 h-20 hover:bg-gray-700 flex items-center
           justify-center material-symbols-outlined text-4xl cursor-pointer 
           ${cell === 'X' ? 'text-pink-500' : 'text-blue-400'}`"
        >
          {{ cell === "X" ? "close" : cell === "O" ? "circle" : "" }}
        </div>
      </div>
    </div>

    <h2 v-if="winner" class="text-6xl font-bold mb-8">ผู้เล่น '{{ winner }}' ชนะ!</h2>
    <h2 v-else-if="isDraw" class="text-6xl font-bold mb-8">เสมอ!</h2>

    <button
      @click="ResetGame"
      class="px-4 py-2 bg-pink-500 rounded uppercase font-bold hover:bg-pink-600 duration-300"
    >
      เริ่มใหม่
    </button>
  </main>
</template>

<style scoped></style>
