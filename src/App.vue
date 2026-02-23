<script setup lang="ts">
import { ref, watch, computed, onUnmounted } from 'vue'

const gridSize = ref(3) // 3x3 to 6x6
const difficulty = ref<'beginner' | 'advanced'>('beginner')

const difficulties = [
  { key: 'beginner' as const, label: '初級', desc: '按過變暗' },
  { key: 'advanced' as const, label: '進階', desc: '按過不變' }
]
const colorMode = ref(false)

// Vibrant color palette for random color mode
const colorPalette = [
  '#f87171', '#fb923c', '#fbbf24', '#a3e635',
  '#34d399', '#22d3ee', '#60a5fa', '#a78bfa',
  '#e879f9', '#fb7185', '#38bdf8', '#4ade80',
  '#facc15', '#f97316', '#c084fc', '#2dd4bf'
]

const randomColor = () => colorPalette[Math.floor(Math.random() * colorPalette.length)]

const currentTarget = ref(1)
const gameState = ref<'idle' | 'playing' | 'finished'>('idle')
const numbers = ref<{ value: number; id: string; isClicked: boolean; shake: boolean; color: string }[]>([])

// Timer state
const timeElapsed = ref(0)
let timerInterval: number | null = null

const formattedTime = computed(() => {
  const totalMs = timeElapsed.value
  const minutes = Math.floor(totalMs / 60000).toString().padStart(2, '0')
  const seconds = Math.floor((totalMs % 60000) / 1000).toString().padStart(2, '0')
  const ms = Math.floor((totalMs % 1000) / 10).toString().padStart(2, '0')
  return `${minutes}:${seconds}.${ms}`
})

// Max number for current grid size
const maxNumber = computed(() => gridSize.value * gridSize.value)

// Grid dynamic class mapping
const gridColsClass = computed(() => {
  const cols: Record<number, string> = {
    3: 'grid-cols-3',
    4: 'grid-cols-4',
    5: 'grid-cols-5',
    6: 'grid-cols-6'
  }
  return cols[gridSize.value] || 'grid-cols-3'
})

// Shuffle array helper
const shuffle = (array: number[]) => {
  const newArr = [...array]
  for (let i = newArr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [newArr[i], newArr[j]] = [newArr[j], newArr[i]]
  }
  return newArr
}

// Generate new grid
const generateGrid = () => {
  const max = maxNumber.value
  const rawNumbers = Array.from({ length: max }, (_, i) => i + 1)
  const shuffled = shuffle(rawNumbers)
  
  numbers.value = shuffled.map((num, i) => ({
    value: num,
    id: `${num}-${i}`,
    isClicked: false,
    shake: false,
    color: randomColor()
  }))
  currentTarget.value = 1
  gameState.value = 'idle'
  timeElapsed.value = 0
  if (timerInterval) clearInterval(timerInterval)
}

// Handle cell click
const handleCellClick = (cell: any) => {
  if (gameState.value === 'finished') return

  if (cell.value === currentTarget.value) {
    // Correct pick
    if (currentTarget.value === 1 && gameState.value === 'idle') {
      startGame()
    }
    
    cell.isClicked = true
    
    if (currentTarget.value === maxNumber.value) {
      finishGame()
    } else {
      currentTarget.value++
    }
  } else if (!cell.isClicked) {
    // Wrong pick logic: trigger shake animation
    cell.shake = true
    setTimeout(() => { cell.shake = false }, 400) // Duration matches CSS animate-shake
  }
}

const startGame = () => {
  gameState.value = 'playing'
  const startTime = Date.now() - timeElapsed.value
  
  timerInterval = window.setInterval(() => {
    timeElapsed.value = Date.now() - startTime
  }, 10)
}

const finishGame = () => {
  gameState.value = 'finished'
  if (timerInterval) clearInterval(timerInterval)
}

const restartGame = () => {
  generateGrid()
}

// Watch grid size change to regenerate
watch(gridSize, () => {
  generateGrid()
}, { immediate: true })

onUnmounted(() => {
  if (timerInterval) clearInterval(timerInterval)
})
</script>

<template>
  <div class="flex h-screen w-full bg-slate-950 text-slate-100 font-sans overflow-hidden">
    
    <!-- Sidebar Area (1/4 width) -->
    <aside class="w-1/4 min-w-[250px] bg-slate-900/80 backdrop-blur-md border-r border-slate-800 p-8 flex flex-col justify-center shadow-xl z-10 relative">
      <div class="absolute top-0 right-0 w-full h-full bg-gradient-to-br from-indigo-500/10 to-transparent pointer-events-none"></div>

      <h1 class="text-3xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-cyan-400 to-indigo-400 mb-10 tracking-wide text-center">
        Schulte Table
      </h1>
      
      <div class="space-y-6 z-10">
        <!-- Size Selector -->
        <div class="flex flex-col space-y-3">
          <label class="text-sm font-semibold text-slate-400 uppercase tracking-wider">Grid Size</label>
          <div class="flex flex-col gap-2">
            <button
              v-for="size in [3, 4, 5, 6]"
              :key="size"
              @click="gridSize = size"
              :disabled="gameState === 'playing'"
              :class="[
                'w-full px-4 py-3 rounded-xl text-sm font-bold tracking-wide transition-all duration-300 border focus:outline-none',
                gridSize === size
                  ? 'bg-cyan-500/20 border-cyan-500/60 text-cyan-300 shadow-[0_0_15px_rgba(34,211,238,0.15)]'
                  : 'bg-slate-800/50 border-slate-700 text-slate-300 hover:bg-slate-700/70 hover:border-slate-500 hover:text-slate-100',
                gameState === 'playing' ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer'
              ]"
            >
              {{ size }} x {{ size }}
            </button>
          </div>
          <p v-if="gameState === 'playing'" class="text-xs text-amber-500/80 mt-1">Cannot change size while playing</p>
        </div>

        <!-- Difficulty Selector -->
        <div class="flex flex-col space-y-3">
          <label class="text-sm font-semibold text-slate-400 uppercase tracking-wider">Difficulty</label>
          <div class="flex flex-col gap-2">
            <button
              v-for="d in difficulties"
              :key="d.key"
              @click="difficulty = d.key; restartGame()"
              :disabled="gameState === 'playing'"
              :class="[
                'w-full px-4 py-3 rounded-xl text-sm font-bold tracking-wide transition-all duration-300 border focus:outline-none flex items-center justify-between',
                difficulty === d.key
                  ? 'bg-cyan-500/20 border-cyan-500/60 text-cyan-300 shadow-[0_0_15px_rgba(34,211,238,0.15)]'
                  : 'bg-slate-800/50 border-slate-700 text-slate-300 hover:bg-slate-700/70 hover:border-slate-500 hover:text-slate-100',
                gameState === 'playing' ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer'
              ]"
            >
              <span>{{ d.label }}</span>
              <span class="text-xs font-normal opacity-60">{{ d.desc }}</span>
            </button>
          </div>
          <p v-if="gameState === 'playing'" class="text-xs text-amber-500/80 mt-1">Cannot change difficulty while playing</p>
        </div>

        <!-- Color Mode Toggle -->
        <div class="flex flex-col space-y-3">
          <label class="text-sm font-semibold text-slate-400 uppercase tracking-wider">Color Mode</label>
          <button
            @click="colorMode = !colorMode; restartGame()"
            :disabled="gameState === 'playing'"
            :class="[
              'w-full px-4 py-3 rounded-xl text-sm font-bold tracking-wide transition-all duration-300 border focus:outline-none flex items-center justify-between',
              colorMode
                ? 'bg-cyan-500/20 border-cyan-500/60 text-cyan-300 shadow-[0_0_15px_rgba(34,211,238,0.15)]'
                : 'bg-slate-800/50 border-slate-700 text-slate-300 hover:bg-slate-700/70 hover:border-slate-500 hover:text-slate-100',
              gameState === 'playing' ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer'
            ]"
          >
            <span>🎨 隨機顏色</span>
            <span class="text-xs font-normal opacity-60">{{ colorMode ? 'ON' : 'OFF' }}</span>
          </button>
        </div>
      </div>
    </aside>

    <!-- Main Game Area (3/4 width) -->
    <main class="flex-1 relative flex flex-col items-center justify-center p-8 bg-[radial-gradient(ellipse_at_top,_var(--tw-gradient-stops))] from-slate-900 via-slate-950 to-black">
      
      <!-- Top Status/Action Panel -->
      <div class="absolute top-8 w-full max-w-3xl flex justify-between items-end px-8 z-10">
        <div class="flex flex-col">
          <span class="text-slate-500 text-sm font-semibold uppercase tracking-widest mb-1 shadow-black drop-shadow-md">Target</span>
          <div class="text-5xl font-black text-cyan-400 drop-shadow-[0_0_15px_rgba(34,211,238,0.4)]">
            {{ gameState === 'finished' ? '-' : currentTarget }}
          </div>
        </div>

        <button 
          @click="restartGame"
          title="Restart Game"
          class="group flex items-center gap-2 px-6 py-3 bg-slate-800 hover:bg-slate-700 border border-slate-700 hover:border-slate-500 rounded-full text-sm font-bold text-slate-300 transition-all duration-300 cursor-pointer shadow-lg hover:shadow-cyan-500/20 active:scale-95"
        >
          <svg class="w-4 h-4 group-hover:-rotate-180 transition-transform duration-500" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path></svg>
          Restart
        </button>
      </div>

      <!-- Timer (Above Grid) -->
      <div class="mb-8 flex flex-col items-center z-10">
        <div 
          class="text-6xl font-mono font-light tabular-nums tracking-tight transition-colors duration-300"
          :class="{
            'text-slate-100 drop-shadow-md': gameState === 'playing' || gameState === 'idle',
            'text-emerald-400 drop-shadow-[0_0_20px_rgba(52,211,153,0.5)] font-medium scale-110 transform transition-transform': gameState === 'finished'
          }"
        >
          {{ formattedTime }}
        </div>
        <div v-if="gameState === 'finished'" class="mt-4 text-emerald-400 font-bold tracking-widest text-lg animate-pulse uppercase">
          Completed!
        </div>
        <div v-else-if="gameState === 'idle'" class="mt-4 text-slate-500 font-medium tracking-wide text-sm opacity-80 animate-pulse">
          Click 1 to Start
        </div>
         <div v-else class="mt-4 text-slate-500 font-medium tracking-wide text-sm opacity-0">
          Playing
        </div>
      </div>

      <!-- Grid Area -->
      <div 
        class="w-full max-w-[600px] aspect-square bg-slate-800/40 rounded-3xl border border-slate-700/50 backdrop-blur-md shadow-2xl p-4 sm:p-6 lg:p-8 z-10 transition-all duration-500"
      >
        <div 
          class="grid gap-2 sm:gap-3 lg:gap-4 w-full h-full"
          :class="gridColsClass"
        >
          <button
            v-for="cell in numbers"
            :key="cell.id"
            @click="handleCellClick(cell)"
            :class="[
              'relative flex items-center justify-center rounded-xl text-4xl sm:text-5xl font-bold transition-all duration-300 border focus:outline-none user-select-none',
              cell.isClicked && difficulty === 'beginner'
                ? 'bg-slate-900/50 border-slate-800 text-slate-600 scale-95 shadow-inner' 
                : cell.isClicked && difficulty === 'advanced'
                  ? 'bg-slate-700/60 border-slate-600 text-slate-200'
                  : 'bg-slate-700/60 border-slate-600 text-slate-200 hover:bg-slate-600/80 hover:-translate-y-1 hover:shadow-lg hover:shadow-cyan-500/20 active:translate-y-0 active:scale-95 cursor-pointer',
              cell.shake ? 'animate-shake bg-red-500/20 border-red-500/50 text-red-100' : ''
            ]"
            :disabled="cell.isClicked"
          >
            <!-- Number Text -->
            <span class="z-10 relative" :style="colorMode && !(cell.isClicked && difficulty === 'beginner') ? { color: cell.color } : {}">{{ cell.value }}</span>
            
            <!-- Glow effect on hover -->
            <div 
              v-if="!cell.isClicked"
              class="absolute inset-0 rounded-xl bg-cyan-400/0 hover:bg-cyan-400/5 transition-colors duration-300"
            ></div>
          </button>
        </div>
      </div>

    </main>

  </div>
</template>

<style scoped>
/* Shake Animation for incorrect clicks */
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px) rotate(-1deg); }
  50% { transform: translateX(5px) rotate(1deg); }
  75% { transform: translateX(-5px) rotate(-1deg); }
}

.animate-shake {
  animation: shake 0.4s cubic-bezier(.36,.07,.19,.97) both;
}

/* Tabular nums for timer to prevent width shifting */
.tabular-nums {
  font-variant-numeric: tabular-nums;
}
</style>
