<template>

<link rel="manifest" href="/manifest.json">
<meta name="theme-color" content="#020617">

  <section class="memory-game">

    <!-- 🟣 START MENU -->
    <div v-if="!started" class="start-screen">

      <div class="menu-box">
        <h1>🧠 Memory Game</h1>
        <p class="subtitle">Find all pairs as fast as you can</p>

        <!-- Difficulty -->
        <div class="menu-row">
          <span>Difficulty</span>
          <div class="segment">
            <button @click="level='easy'" :class="{active: level==='easy'}">Easy</button>
            <button @click="level='hard'" :class="{active: level==='hard'}">Hard</button>
          </div>
        </div>

        <!-- Theme -->
        <div class="menu-row">
          <span>Theme</span>
          <div class="segment">
            <button @click="theme='emoji'" :class="{active: theme==='emoji'}">Emoji</button>
            <button @click="theme='icons'" :class="{active: theme==='icons'}">Icons</button>
          </div>
        </div>

        <button class="start-btn" @click="startGame(level)">
          Start Game
        </button>
      </div>

    </div>

    <!-- 🎮 GAME -->
    <div v-else>

      <h1 class="title">🧠 Memory Game</h1>

      <!-- STATS -->
      <div class="stats">
        <span>⏱ {{ time }}s</span>
        <span>🎯 {{ moves }}</span>
        <span>🏆 {{ bestScore || '-' }}</span>
      </div>

      <!-- GRID -->
      <div class="grid" :class="level">
        <div 
          v-for="card in cards"
          :key="card.id"
          class="card"
          :class="{ flipped: card.flipped || card.matched }"
          @click="flipCard(card)"
        >
          <div class="inner">
            <div class="front">❓</div>
            <div class="back">{{ card.content }}</div>
          </div>
        </div>
      </div>

      <!-- 🎉 WIN SCREEN -->
      <div v-if="isWin" class="win-overlay">

        <div class="win-box">
          <h2>🎉 You Win!</h2>

          <div class="stars">
            <span v-for="i in 3" :key="i">
              {{ i <= stars ? '⭐' : '☆' }}
            </span>
          </div>

          <p>{{ time }} sec • {{ moves }} moves</p>

          <button class="play-btn" @click="startGame(level)">
            Play Again
          </button>
        </div>

      </div>

      <!-- CONFETTI -->
      <div v-if="isWin" class="confetti"></div>

    </div>

  </section>
</template>

<script setup>
import { ref, computed } from 'vue'

const clickSound = new Audio('/click.mp3')
const winSound = new Audio('/win.mp3')
const errorSound = new Audio('/error.mp3')
const flipSound = new Audio('/flip.mp3')

const emojiTheme = ['🍕','🚀','🎧','💻','🔥','🎮','🌈','⚡','🍓','🐱','🎲','🧩']
const iconTheme = ['⭐','❤️','⚽','🎵','📱','💎','🔔','📸','🎁','🚗','🌙','☀️']

const theme = ref('emoji')
const started = ref(false)

const cards = ref([])
const flippedCards = ref([])
const moves = ref(0)
const time = ref(0)
const level = ref('easy')

const bestScore = ref(localStorage.getItem('bestScore') || null)

let timer = null

const stars = computed(() => {
  if (moves.value <= 10) return 3
  if (moves.value <= 18) return 2
  return 1
})

const startGame = (lvl) => {
  started.value = true
  level.value = lvl

  const source = theme.value === 'emoji' ? emojiTheme : iconTheme
  const count = lvl === 'easy' ? 6 : 12

  const selected = source.slice(0, count)

  const doubled = [...selected, ...selected]
    .sort(() => Math.random() - 0.5)
    .map((item, i) => ({
      id: i,
      content: item,
      flipped: false,
      matched: false
    }))

  cards.value = doubled
  flippedCards.value = []
  moves.value = 0
  time.value = 0

  clearInterval(timer)
  timer = setInterval(() => time.value++, 1000)
}

const flipCard = (card) => {
  if (card.flipped || card.matched || flippedCards.value.length === 2) return

  navigator.vibrate?.(30)

  flipSound.currentTime = 0
  flipSound.play()

  clickSound.currentTime = 0
  clickSound.play()

  card.flipped = true
  flippedCards.value.push(card)

  if (flippedCards.value.length === 2) {
    moves.value++
    checkMatch()
  }
}

const checkMatch = () => {
  const [a, b] = flippedCards.value

  if (a.content === b.content) {
    a.matched = true
    b.matched = true
    flippedCards.value = []
  } else {
    errorSound.currentTime = 0
    errorSound.play()

    setTimeout(() => {
      a.flipped = false
      b.flipped = false
      flippedCards.value = []
    }, 700)
  }
}

const isWin = computed(() => {
  if (!cards.value.length) return false

  const win = cards.value.every(c => c.matched)

  if (win) {
    clearInterval(timer)

    winSound.currentTime = 0
    winSound.play()

    if (!bestScore.value || time.value < bestScore.value) {
      bestScore.value = time.value
      localStorage.setItem('bestScore', time.value)
    }
  }

  return win
})
</script>

<style scoped>
.memory-game {
  text-align: center;
  min-height: 100vh;
  background: radial-gradient(circle at top, #0f172a, #020617);
  color: white;
}

/* MENU */
.start-screen {
  position: fixed;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.menu-box {
  background: rgba(255,255,255,0.05);
  backdrop-filter: blur(20px);
  padding: 30px;
  border-radius: 18px;
  width: 260px;
}

.subtitle {
  opacity: 0.6;
  font-size: 14px;
}

/* ROW */
.menu-row {
  margin-top: 15px;
  text-align: left;
  font-size: 14px;
}

/* SEGMENT */
.segment {
  margin-top: 6px;
  display: flex;
  background: rgba(255,255,255,0.06);
  border-radius: 8px;
  padding: 3px;
}

.segment button {
  flex: 1;
  padding: 6px;
  font-size: 13px;
  border: none;
  background: transparent;
  color: #aaa;
  cursor: pointer;
  border-radius: 6px;
}

.segment button.active {
  background: rgba(255,255,255,0.15);
  color: white;
}

/* START BUTTON */
.start-btn {
  margin-top: 20px;
  width: 100%;
  padding: 12px;
  border-radius: 10px;
  border: none;
  background: #6366f1;
  color: white;
  cursor: pointer;
}

/* GRID */
.grid {
  margin-top: 30px;
  display: grid;
  justify-content: center;
}

.grid.easy {
  grid-template-columns: repeat(4, 80px);
  gap: 14px;
}

.grid.hard {
  grid-template-columns: repeat(6, 65px);
  gap: 16px;
}

/* CARD */
.card {
  width: 100%;
  height: 80px;
  perspective: 800px;
}

.inner {
  width: 100%;
  height: 100%;
  transition: 0.5s;
  transform-style: preserve-3d;
}

.card.flipped .inner {
  transform: rotateY(180deg);
}

.front,
.back {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  backface-visibility: hidden;
}

.front {
  background: #7c3aed;
}

.back {
  background: #020617;
  transform: rotateY(180deg);
}

/* WIN */
.win-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.6);
  display: flex;
  align-items: center;
  justify-content: center;
}

.win-box {
  background: #020617;
  padding: 25px;
  border-radius: 16px;
  text-align: center;
}

.play-btn {
  margin-top: 15px;
  padding: 12px 20px;
  border-radius: 10px;
  border: none;
  background: #22c55e;
  color: black;
  cursor: pointer;
}

/* CONFETTI */
.confetti {
  position: fixed;
  inset: 0;
  pointer-events: none;
  background-image: radial-gradient(circle, #22c55e 2px, transparent 2px);
  background-size: 20px 20px;
  animation: confetti 1s linear infinite;
}

@keyframes confetti {
  from { transform: translateY(-100%); }
  to { transform: translateY(100%); }
}
</style>
