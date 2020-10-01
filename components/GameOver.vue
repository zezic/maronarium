<template>
<div class="game-over">
  Game Over
  <div class="hint">Session Maximum:</div>
  <div class="balance"><span class="prefix">0x</span>{{ hexBalance }}</div>
</div>
</template>

<script>
export default {
  props: {
    maxBalance: {
      type: Number,
      default: 0
    }
  },
  computed: {
    hexBalance () {
      return this.maxBalance.toString(16).toUpperCase()
    }
  },
  methods: {
    engageKeys () {
      document.addEventListener('keydown', this.handleKeyDown)
    },
    disengageKeys () {
      document.removeEventListener('keydown', this.handleKeyDown)
    },
    handleKeyDown (event) {
      if (event.keyCode === 32) { // space
        this.$emit('onReturn')
      }
    }
  },
  mounted () {
    this.engageKeys()
  },
  beforeDestroy () {
    this.disengageKeys()
  }
}
</script>

<style scoped>
.game-over {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-size: 7vw;
  color: #cecece;
  background-color: rgba(12, 12, 12, .95);
}

.hint {
  margin-top: 10vw;
  font-size: 3.5vw;
}

.balance {
  font-size: 10vw;
}

.balance .prefix {
  opacity: .25;
}

@media screen and (min-width: 640px) {
  .game-over {
    font-size: 18px;
  }

  .hint {
    margin-top: 20px;
    font-size: 10px;
  }

  .balance {
    margin-top: 4px;
    font-size: 24px;
  }
}
</style>