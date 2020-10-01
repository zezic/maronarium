<template>
<div class="results" @click='handleClick'>
  <div class="colors">
    <color-drop v-for='(color, idx) in colors' :rgb='color' :key='idx'/>
  </div>
  <div class="graphs-and-earnings">
    <div class='graphs'>
      <hue-circle class="hue-circle" :colors='colors'/>
    </div>
    <div class="earnings">
      <div class="raw-summ">Raw summ: 0x{{ rawSummHex }}</div>
      <div class="total">{{ total > 0 ? '+' : '-' }}<span class="prefix">0x</span>{{ total ? totalHex : rawSummHex }}{{ total > 0 ? '!' : ' :(' }}</div>
    </div>
  </div>
  <harmonies class="harmonies" :colors='colors' @multiplierUpdate='handleMultUpdate'/>
</div>
</template>

<script>
import ColorDrop from '~/components/ColorDrop'
import HueCircle from '~/components/HueCircle'
import Harmonies from '~/components/Harmonies'

export default {
  components: {
    ColorDrop,
    HueCircle,
    Harmonies
  },
  props: {
    colors: {
      type: Array,
      required: true
    },
    active: {
      type: Boolean,
      default: true
    }
  },
  data: () => ({
    mult: 0
  }),
  methods: {
    handleMultUpdate (mult) {
      this.mult = mult
    },
    handleClick () {
      this.$emit('onReturn', this.total || -this.rawSumm)
    },
    engageKeys () {
      document.addEventListener('keydown', this.handleKeyDown)
    },
    disengageKeys () {
      document.removeEventListener('keydown', this.handleKeyDown)
    },
    handleKeyDown (event) {
      if (this.active && event.keyCode === 32) { // space
        this.handleClick()
      }
    }
  },
  computed: {
    rawSumm () {
      return this.colors.reduce((acc1, color) => {
        return acc1 + color.reduce((acc2, int) => acc2 + int, 0)
      }, 0)
    },
    rawSummHex () {
      return this.rawSumm.toString(16).toUpperCase()
    },
    total () {
      return this.rawSumm * this.mult
    },
    totalHex () {
      return this.total.toString(16).toUpperCase()
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
.results {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  cursor: pointer;
}

.colors {
  display: flex;
  justify-content: center;
  padding-top: 5vw;
}

@media screen and (min-width: 640px) {
  .results {
    overflow: auto;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .colors {
    padding: 0;
    padding-top: 12px;
    width: 240px;
  }
}

.graphs {
  display: flex;
  justify-content: center;
}

@media screen and (min-width: 640px) {
  .graphs {
    margin-top: 8px;
  }
}

.earnings {
  color: #e5e5e5;
  text-align: center;
}

.raw-summ {
  font-size: 3.5vw;
  font-weight: 700;
  opacity: .25;
  margin: 7vw 0 2vw;
}

.total {
  font-size: 12vw;
}

@media screen and (min-width: 640px) {
  .raw-summ {
    font-size: 12px;
    margin: 16px 0 8px;
  }

  .total {
    font-size: 24px;
  }

  .graphs-and-earnings {
    margin: 0 36px;
  }
}

.total .prefix {
  opacity: .2;
}

.harmonies {
  margin: 3vw 10vw;
}
@media screen and (min-width: 640px) {
  .harmonies {
    width: 240px;
    margin: 0;
    height: 140px;
  }
}
</style>