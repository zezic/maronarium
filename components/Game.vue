<template>
<div
  class="game"
  @touchstart='handleTouchStart'
  @touchend='handleTouchEnd'
  @touchcancel='handleTouchCancel'
  @touchmove='handleTouchMove'
>
  <div class="collected">
    <color-bar class='color' v-for='(color, idx) in collected' :key='idx' :rgb='color'/>
  </div>
  <div class="ground">
    <cell
      v-for='(cell, idx) in cells'
      :label='cell'
      :key='idx'
      :prehint='prehint'
    />
    <div class='decoration'>
      <div class='piece top'></div>
      <div class='piece bottom'></div>
    </div>
    <div class="arrow" :class='{ active: complete }'></div>
    <div class='dot-wrapper' :style='dotWrapperStyle'>
      <div class="dot"></div>
    </div>
  </div>
  <div class="stats">
    <div class="stat">
      <div class="label">Score:</div>
      <div class="value"><span class="prefix">0x</span>{{ balanceHex }}</div>
    </div>
    <button class='button' @click='toggleFullscreen'>Fullscreen</button>
  </div>
</div>
</template>

<script>
import chroma from 'chroma-js'
import nearestColor from 'nearest-color'
import namedColors from 'color-name-list'

import Vue from 'vue'
import Cell from '~/components/Cell'
import ColorBar from '~/components/ColorBar'

const colors = namedColors.reduce((o, { name, hex }) => Object.assign(o, { [name]: hex }), {})
const nearest = nearestColor.from(colors)

const fieldSize = 3
const groundSize = 1 + fieldSize + 1

function coordToIdx ([x, y]) {
  return y * groundSize + x
}

function idxToCoord (idx) {
  return [idx % groundSize, Math.floor(idx / groundSize)]
}

function coordInField ([x, y]) {
  const fieldX = x - 1
  const fieldY = y - 1
  return 0 <= fieldX && fieldX < fieldSize && 0 <= fieldY && fieldY < fieldSize
}

function generateGround () {
  const output = []
  for (let i = 0; i < groundSize * groundSize; i++) {
    if (coordInField(idxToCoord(i))) {
      output.push(Math.round(Math.random() * 255))
    } else {
      output.push(undefined)
    }
  }
  return output
}

let swipeFrom = [0, 0]
let activeTouchId = null

let synth = null
let sampler = null
let toneIsReady = false

let moveSounds = []
let chordSounds = []

let currentMoveIdx = -1

function playChord (idx) {
  chordSounds[idx].pause()
  chordSounds[idx].currentTime = 0
  chordSounds[idx].play()
}

function playMove () {
  currentMoveIdx = (currentMoveIdx + 1) % 3
  moveSounds[currentMoveIdx].pause()
  moveSounds[currentMoveIdx].currentTime = 0
  moveSounds[currentMoveIdx].play()
}

export default {
  components: {
    Cell,
    ColorBar
  },
  props: {
    active: {
      type: Boolean,
      default: true
    },
    balance: {
      type: Number,
      default: 0
    }
  },
  data: () => ({
    cells: [...new Array(groundSize * groundSize)].map(_ => undefined),
    dotPosition: [0, 2],
    collected: [
      [],
      [],
      []
    ],
  }),
  computed: {
    dotWrapperStyle () {
      const cellSize = 100 / groundSize // 100% / n of cells
      const halfCell = cellSize / 2
      const offsets = this.dotPosition.map((comp) => {
        return halfCell + comp * cellSize
      })
      offsets[0] = Math.max(offsets[0], 13)
      return {
        transform: `translate(${offsets[0]}%, ${offsets[1]}%)`
      }
    },
    complete () {
      return this.collected.reduce((acc, rgb) => acc +  rgb.length, 0) === 9
    },
    balanceHex () {
      return Math.abs(this.balance).toString(16).toUpperCase()
    },
    prehint () {
      return this.collected.find(color => color.length === 2)
    }
  },
  methods: {
    handleTouchStart (event) {
      swipeFrom = [
        event.changedTouches[0].screenX,
        event.changedTouches[0].screenY
      ]
      activeTouchId = event.changedTouches[0].identifier
    },
    handleTouchEnd () {
      this.swipeDelta = [0, 0]
      this.activeTouchId = null
    },
    handleTouchCancel () {
      this.swipeDelta = [0, 0]
      this.activeTouchId = null
    },
    handleTouchMove (event) {
      if (activeTouchId === null) { return }
      for (let i = 0; i < event.changedTouches.length; i++) {
        const touch = event.changedTouches[i]
        if (touch.identifier === activeTouchId) {
          const newScreenCoords = [touch.screenX, touch.screenY]
          const deltas = swipeFrom.map((coord, idx) => {
            return newScreenCoords[idx] - coord
          })
          this.swipeDelta = deltas
          const absolutes = deltas.map(delta => Math.abs(delta))
          for (let a = 0; a < absolutes.length; a++) {
            const absolute = absolutes[a]
            const delta = deltas[a]
            if (absolute > 15) {
              const otherIdx = a === 0 ? 1 : 0
              const otherAbs = absolutes[otherIdx]
              if (absolute / otherAbs > 3) {
                activeTouchId = null
                this.swipeDelta = [0, 0]
                if (a === 0) {
                  this.swipeHandler(delta > 0 ? 'right' : 'left')
                } else {
                  this.swipeHandler(delta > 0 ? 'down' : 'up')
                }
              }
            }
          }
          break
        }
      }
    },
    handleKeyDown (event) {
      if (!this.active) { return }
      const code = event.keyCode
      if (code >= 37 && code <= 40) {
        const directionIdx = code - 37
        this.move(directionIdx)
      }
    },
    move (directionIdx) {
      const directions = [
        [ -1,  0 ],
        [  0, -1 ],
        [  1,  0 ],
        [  0,  1 ]
      ]
      const direction = directions[directionIdx]
      const target = this.dotPosition.map((comp, idx) => {
        return comp + direction[idx]
      })
      if (this.complete) {
        const [x, y] = target
        if (x === groundSize - 1 && y === Math.floor(groundSize / 2)) {
          playMove()
          this.dotPosition = target
          this.$emit('complete', this.collected)
          return
        }
      }
      if (coordInField(target)) {
        playMove()
        this.dotPosition = target
      }
    },
    engageKeys () {
      document.addEventListener('keydown', this.handleKeyDown)
    },
    disengageKeys () {
      document.removeEventListener('keydown', this.handleKeyDown)
    },
    consumeHex (hex) {
      const lastColor = this.collected.find(color => color.length < 3)
      lastColor.push(hex)

      if (lastColor.length === 3) {
        playChord(this.collected.indexOf(lastColor))
      }
    },
    fillCells () {
      this.cells = generateGround()
    },
    swipeHandler (event) {
      if (!this.active) { return }
      const swipeDirectionsIndexed = [
        'left', 'up', 'right', 'down'
      ]
      this.move(swipeDirectionsIndexed.indexOf(event))
    },
    reset () {
      this.fillCells()
      this.dotPosition = [0, 2]
      this.collected = [[], [], []]
    },
    toggleFullscreen () {
      if (!document.fullscreenElement) {
        document.documentElement.requestFullscreen()
      } else {
        if (document.exitFullscreen) {
          document.exitFullscreen()
        }
      }
    }
  },
  watch: {
    dotPosition (newPosition) {
      const newIdx = coordToIdx(newPosition)
      const contents = this.cells[newIdx]
      if (contents !== null && contents !== undefined) {
        this.consumeHex(this.cells[newIdx])
        Vue.set(this.cells, newIdx, null)
      }
    }
  },
  mounted () {
    if (chordSounds.length === 0) {
      const sampleNames = [
        '/samples/chord1.mp3',
        '/samples/chord2.mp3',
        '/samples/chord3.mp3'
      ]
      chordSounds = sampleNames.map(sampleName => new Audio(sampleName))
    }
    if (moveSounds.length === 0) {
      const sampleNames = [
        '/samples/move1.mp3',
        '/samples/move2.mp3',
        '/samples/move3.mp3'
      ]
      moveSounds = sampleNames.map(sampleName => new Audio(sampleName))
    }
    this.fillCells()
    this.engageKeys()
  },
  beforeDestroy () {
    this.disengageKeys()
  }
}
</script>

<style scoped>
.game {
  min-height: 100vh;
  width: 100vw;
  overflow: hidden;
}

@media screen and (min-width: 640px) {
  .game {
    display: flex;
    align-items: flex-start;
    justify-content: center;
    flex-wrap: wrap;
  }
}

.collected {
  padding: 9vw;
  padding-bottom: 0;
}

.collected .color:not(:last-child) {
  margin-bottom: 3vw;
}

@media screen and (min-width: 640px) {
  .collected {
    padding: 40px;
    padding-right: 0;
    width: 320px;
    height: 320px;
    margin-top: auto;
  }
  .collected .color:not(:last-child) {
    margin-bottom: 10px;
  }
}

.ground {
  width: 110vw;
  height: 110vw;
  margin: -5vw;
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  grid-template-rows: repeat(5, 1fr);
  position: relative;
  overflow: hidden;
  pointer-events: none;
}

@media screen and (min-width: 640px) {
  .ground {
    width: 320px;
    height: 320px;
    margin: 0;
    flex-shrink: 0;
    margin-top: auto;
  }
}

.decoration {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.piece {
  height: 37%;
  margin: 12.5%;
  border: 0.6vw solid #2a2a2a;
  border-radius: 12vw;
}

@media screen and (min-width: 640px) {
  .piece {
    border: 3px solid #2a2a2a;
    border-radius: 36px;
  }
}

.piece.top {
  border-bottom-left-radius: 0;
  border-bottom-right-radius: 0;
  border-bottom-width: 0;
}

.piece.bottom {
  border-top-left-radius: 0;
  border-top-right-radius: 0;
  border-top-width: 0;
}

.dot-wrapper {
  position: absolute;
  width: 100%;
  height: 100%;
  transition: transform .2s ease;
  will-change: transform;
}

.dot {
  position: absolute;
  width: 24px;
  height: 24px;
  left: -12px;
  top: -12px;
  background-color: #e5e5e5;
  border-radius: 50%;
}

.arrow {
  position: absolute;
  right: 8vw;
  top: 50%;
  margin-top: -6vw;
  height: 12vw;
  width: 12vw;
  display: flex;
  align-items: center;
  justify-content: center;
}

.arrow::before {
  display: block;
  content: ' ';
  position: absolute;
  background-color: #2a2a2a;
  height: 0.6vw;
  width: 80%;
}

.arrow::after {
  display: block;
  content: ' ';
  position: absolute;
  border: 0.6vw solid #2a2a2a;
  height: 40%;
  width: 40%;
  transform: translateX(40%) rotate(45deg);
  border-bottom-width: 0;
  border-left-width: 0;
}

@media screen and (min-width: 640px) {
  .arrow {
    right: 16px;
    margin-top: -24px;
    height: 48px;
    width: 48px;
  }

  .arrow::before {
    height: 3px;
  }

  .arrow::after {
    border: 3px solid #2a2a2a;
    border-bottom-width: 0;
    border-left-width: 0;
  }
}

.arrow.active::before {
  background-color: #e5e5e5;
}

.arrow.active::after {
  border-color: #e5e5e5;
}

.stats {
  padding: 9vw;
  padding-top: 0;
  padding-bottom: 0;
  display: flex;
}

.stats .stat {
  color: #e5e5e5;
  margin-right: auto;
}

.stats .stat .label {
  opacity: .25;
  font-size: 4vw;
}

.stats .stat .value {
  font-size: 10vw;
}

.stats .stat .value .prefix {
  opacity: .25;
}

.button {
  background-color: transparent;
  border: 0.6vw solid #2a2a2a;
  border-radius: 2vw;
  color: #e5e5e5;
  font-family: 'Anka/Coder';
  padding: 3vw 5vw;
  font-size: 4vw;
}

@media screen and (min-width: 640px) {
  .stats {
    flex-basis: 100%;
    margin-bottom: auto;
    margin-top: -80px;
  }

  .stats .stat {
    width: 640px;
    margin: 0 auto;
    text-align: left;
    padding-left: 44px;
  }

  .stats .button {
    display: none;
  }

  .stats .stat .label {
    font-size: 12px;
  }

  .stats .stat .value {
    font-size: 24px;
  }
}
</style>