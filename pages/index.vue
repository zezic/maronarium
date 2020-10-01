<template>
<div class="index-page">
  <div class='stages'>
    <game
      @complete='handleComplete'
      class='game'
      :class='{ shifted: stage !== "game" }'
      :active='stage === "game"'
      :balance='balance'
      ref='game'
    />
    <results
      :colors='collected'
      class='results'
      :class='{ shifted: stage !== "game" }'
      :active='stage === "results"'
      @onReturn='returnToGame'
    />
  </div>

  <transition name="fade">
    <game-over
      v-if='stage === "game-over"'
      :maxBalance='maxBalance'
      class="game-over"
      @click.native='returnFromOver'
      @onReturn='returnFromOver'
    />
  </transition>
</div>
</template>

<script>
import Game from '~/components/Game'
import GameOver from '~/components/GameOver'
import Results from '~/components/Results'

export default {
  components: {
    Game,
    GameOver,
    Results
  },
  data: () => ({
    stage: 'game',
    collected: [[], [], []],
    balance: 0,
    maxBalance: 0
  }),
  methods: {
    handleComplete (collected) {
      this.collected = collected
      this.stage = 'results'
      setTimeout(() => {
        this.$refs.game.reset()
      }, 500)
    },
    returnToGame (earnings) {
      const newBalance = this.balance + earnings
      this.maxBalance = Math.max(newBalance, this.maxBalance)
      this.balance = newBalance
      
      if (this.balance < 0) {
        this.stage = 'game-over'
        this.balance = 0
      } else {
        this.stage = 'game'
      }
    },
    returnFromOver () {
      this.stage = 'game'
      this.maxBalance = 0
    },
  }
}
</script>

<style>
.index-page {
  overflow-x: hidden;
}

.stages {
  display: flex;
}

.stages > * {
  flex-shrink: 0;
}

.game,
.results {
  transition: transform .2s ease;
}

.results.shifted,
.game.shifted {
  transform: translateX(-100%);
}

.game-over {
  position: absolute;
  left: 0;
  top: 0;
  height: 100vh;
  width: 100vw;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 1s ease;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
</style>
