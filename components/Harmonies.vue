<template>
<div class="harmonies">
  <div class="harm" :class="{ active: analog }">
    <img src='~/assets/images/harmony-analog.svg'>
    <div class="info">
      <div class="name">Analog</div>
      <div class="mult">X3</div>
    </div>
  </div>
  <div class="harm" :class="{ active: triad }">
    <div class="info">
      <div class="name">Triad</div>
      <div class="mult">X4</div>
    </div>
    <img src='~/assets/images/harmony-triad.svg'>
  </div>
  <div class="harm" :class="{ active: sideComp }">
    <img src='~/assets/images/harmony-side-comp.svg'>
    <div class="info">
      <div class="name">Side<br>Comp</div>
      <div class="mult">X1</div>
    </div>
  </div>
  <div class="harm" :class="{ active: splitComp }">
    <div class="info">
      <div class="name">Split<br>Comp</div>
      <div class="mult">X2</div>
    </div>
    <img src='~/assets/images/harmony-split-comp.svg'>
  </div>
</div>
</template>

<script>
import chroma from 'chroma-js'

function minDist (d1, d2) {
  return Math.min(
    Math.abs(d1 - d2),
    Math.abs(d1 - (d2 - 360)),
    Math.abs(d1 - (d2 + 360))
  )
}

function isNear (d1, d2, tolerance) {
  const minDistance = minDist(d1, d2)
  return minDistance <= tolerance 
}

function angleDifference (a1, a2) {
  let diff = (a2 - a1 + 180) % 360 - 180
  return diff < -180 ? diff + 360 : diff
}

export default {
  props: {
    colors: {
      type: Array,
      default: [[], [], []]
    }
  },
  computed: {
    hues () {
      return this.colors.filter(color => {
        return color.length === 3
      }).map(rgb => {
        return chroma(rgb.slice()).get('hsl.h')
      })
    },
    analog () {
      if (this.hues.length !== 3) { return false }
      return this.hues.map((hue, idx) => {
        const [other1, other2] = this.hues.filter((_, oIdx) => oIdx !== idx)
        return (
          (Math.abs(angleDifference(hue, other1) + angleDifference(hue, other2)) < 10) &&
          (Math.abs(angleDifference(hue, other1)) < 110 && Math.abs(angleDifference(hue, other2)) < 110)
        )
      }).filter(symmetry => symmetry).length > 0
    },
    triad () {
      if (this.hues.length !== 3) { return false }
      return this.hues.map((hue, idx) => {
        const [other1, other2] = this.hues.filter((_, oIdx) => oIdx !== idx)
        return (
          (Math.abs(angleDifference(hue, other1) + angleDifference(hue, other2)) < 20) &&
          (Math.abs(angleDifference(hue, other1)) > 30)
        )
      }).filter(symmetry => symmetry).length === 3
    },
    sideComp () {
      if (this.hues.length !== 3) { return false }
      return this.hues.map((hue, idx) => {
        const [other1, other2] = this.hues.filter((_, oIdx) => oIdx !== idx)
        return isNear(hue + 180, other1, 7) || isNear(hue + 180, other2, 7)
      }).filter(hasComp => hasComp).length > 0
    },
    splitComp () {
      if (this.hues.length !== 3) { return false }
      return this.hues.map((hue, idx) => {
        const [other1, other2] = this.hues.filter((_, oIdx) => oIdx !== idx)
        return (
          (Math.abs(angleDifference(hue, other1) + angleDifference(hue, other2)) < 10) &&
          (Math.abs(angleDifference(hue, other1)) > 130 && Math.abs(angleDifference(hue, other2)) > 130)
        )
      }).filter(symmetry => symmetry).length > 0
    },
    multiplier () {
      let mults = []
      if (this.analog) { mults.push(3) }
      if (this.triad) { mults.push(4) }
      if (this.sideComp) { mults.push(1) }
      if (this.splitComp) { mults.push(2) }
      if (mults.length > 0) {
        return mults.reduce((acc, mult) => acc * mult, 1)
      } else {
        return 0
      }
    }
  },
  watch: {
    multiplier (newMult) {
      this.$emit('multiplierUpdate', newMult)
    }
  }
}
</script>

<style scoped>
.harmonies {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
}

.harm {
  display: flex;
  justify-content: center;
  margin: 5vw 0;
}

.harm img {
  width: 15vw;
}

@media screen and (min-width: 640px) {
  .harm {
    margin: 12px 0;
  }

  .harm img {
    width: 48px;
  }
}

.harm.active img {
  filter: invert(100%) brightness(110%);
}

.harm .info {
  width: 20vw;
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  color: #2a2a2a;
}

.harm.active .info {
  color: #e5e5e5;
}

.harm .info .name {
  font-size: 3.5vw;
}
.harm .info .mult {
  font-size: 7vw;
}

@media screen and (min-width: 640px) {
  .harm .info {
    width: 72px;
  }

  .harm .info .name {
    font-size: 12px;
  }
  .harm .info .mult {
    font-size: 18px;
  }
}
</style>