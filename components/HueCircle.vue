<template>
<div class='hue-circle'>
  <div class='bg'><div class='inner'></div></div>
  <svg width='100%' height='100%' viewBox='-1.45 -1.45 2.9 2.9'>
    <defs>
      <linearGradient
        v-for='(line, idx) in lines'
        :key='`gr-${idx}`'
        :id='`gradient-${idx}`'
        :x1='line.gx1'
        :y1='line.gy1'
        :x2='line.gx2'
        :y2='line.gy2'
      >
        <stop offset='0%' :stop-color='line.color1'/>
        <stop offset='100%' :stop-color='line.color2'/>
      </linearGradient>
    </defs>
    <line
      v-for='(line, idx) in lines'
      :key='`l-${idx}`'
      :x1='line.x1'
      :y1='line.y1'
      :x2='line.x2'
      :y2='line.y2'
      :stroke='`url(#gradient-${idx})`'
      stroke-width='.05'
    />
    <circle
      v-for='(circle, idx) in circles'
      :key='`c-${idx}`'
      :cx='circle.x'
      :cy='circle.y'
      r='.2'
      :stroke='circle.color'
      stroke-width='.05'
      fill='#333333'
    ></circle>
  </svg>
</div>
</template>

<script>
import chroma from 'chroma-js'

export default {
  props: {
    colors: {
      type: Array,
      required: true
    }
  },
  computed: {
    circles () {
      return this.colors.filter(color => {
        return color.length === 3
      }).map(rgb => {
        const chromaInstance = chroma(rgb.slice())
        const angle = (chromaInstance.get('hsl.h') - 90) / 360 * Math.PI * 2
        return {
          x: Math.cos(angle),
          y: Math.sin(angle),
          color: chromaInstance.hex()
        }
      })
    },
    lines () {
      return this.circles.map(({x, y, color}, idx) => {
        const otherIdx = (idx + 1) % this.circles.length
        const other = this.circles[otherIdx]
        return {
          x1: x,
          y1: y,
          x2: other.x,
          y2: other.y,
          gx1: x > other.x ? '100%' : '0%',
          gy1: y > other.y ? '100%' : '0%',
          gx2: x > other.x ? '0%' : '100%',
          gy2: y > other.y ? '0%' : '100%',
          color1: color,
          color2: other.color
        }
      })
    },
    // gradients () {
    //   return this.lines.map(({ x1, y1, x2, y2, color1, color2}) => {
        
    //   })
    // }
  }
}
</script>

<style scoped>
.hue-circle {
  width: 38vw;
  height: 38vw;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

@media screen and (min-width: 640px) {
  .hue-circle {
    width: 140px;
    height: 140px;
  }
}

.bg {
  width: 98%;
  height: 98%;
  border: 0.6vw solid #2a2a2a;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.inner {
  width: 46%;
  height: 46%;
  border: 0.6vw solid #2a2a2a;
  border-radius: 50%;
}

@media screen and (min-width: 640px) {
  .bg {
    border: 2px solid #2a2a2a;
  }
  .inner {
    border: 2px solid #2a2a2a;
  }
}

svg {
  position: absolute;
}
</style>