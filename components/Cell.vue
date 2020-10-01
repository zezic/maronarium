<template>
<div
  class="cell"
  :class='{ transparent: label === undefined, dimmed: label === null }'
  :style='style'
>
  <transition name="fade">
    <div class='label' v-if='label !== null && label !== undefined'>{{ hexView }}</div>
  </transition>
</div>
</template>

<script>
import chroma from 'chroma-js'

export default {
  props: {
    label: {
      default: undefined
    },
    prehint: {
      default: undefined
    }
  },
  computed: {
    hexView () {
      if (this.label === null || this.label === undefined) { return '' }
      return this.label.toString(16).toUpperCase().padStart(2, '0')
    },
    style () {
      if (this.label === null || this.label === undefined) { return {} }
      if (!this.prehint || this.prehint.length !== 2) { return {} }
      const chromaInstance = chroma([...this.prehint, this.label])
      const hexString = chromaInstance.hex()
      const luminance = chromaInstance.luminance()
      return {
        backgroundColor: hexString,
        color: chromaInstance.set('hsl.l', chromaInstance.get('hsl.l') + (luminance < 0.5 ? 0.4 : -0.4)).hex()
      }
    }
  }
}
</script>

<style scoped>
.cell {
  border-radius: 2vw;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 4px;
  background-color: #2a2a2a;
  color: #e5e5e5;
  transition: background-color .2s ease;
}

.cell.dimmed {
  opacity: .5;
}

.cell.transparent {
  background-color: transparent;
}

.label {
  font-size: 7.25vw;
}

@media screen and (min-width: 640px) {
  .cell {
    border-radius: 4px;
  }

  .label {
    font-size: 24px;
  }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .2s ease, transform .2s ease;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
  transform: scale(0);
}
</style>