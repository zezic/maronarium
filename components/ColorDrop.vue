<template>
<div class='group'>
  <div class='color' :style="style">
  </div>
  <div class="hex-label">
    <span class="prefix">0x</span><span class="hex-string">{{ hexLabel }}</span><span class="postfix">{{ postfix }}</span>
  </div>
  <div class='name' v-if='name'>{{ name }}</div>
</div>
</template>

<script>
import ColorMixin from '~/components/ColorMixin'

export default {
  mixins: [ColorMixin]
}
</script>

<style scoped>
.group {
  width: 28vw;
  font-size: 3.5vw;
  padding: 6vw 3vw;
}

@media screen and (min-width: 640px) {
  .group {
    width: 80px;
    font-size: 11px;
    padding: 12px 6px;
  }
}

.color {
  margin: 0 auto;
  display: flex;
  height: 16vw;
  width: 16vw;
  border-radius: 8vw;
  align-items: center;
  justify-content: space-between;
  transition: background-color ease .2s;
  background-color: #2a2a2a;
  line-height: 0;
  margin-bottom: 3vw;
}

@media screen and (min-width: 640px) {
  .color {
    height: 48px;
    width: 48px;
    border-radius: 24px;
    margin-bottom: 12px;
    padding: 0;
  }
}

.hex-label {
  color: #e5e5e5;
  font-weight: 700;
  text-align: center;
  margin-bottom: 2vw;
}

@media screen and (min-width: 640px) {
  .hex-label {
    margin-bottom: 4px;
  }
}

.hex-label .prefix,
.hex-label .postfix {
  opacity: .15;
  font-weight: 400;
}

.hex-label > * {
  vertical-align: top;
}

.name {
  text-align: center;
  white-space: normal;
  line-height: 1.25;
  color: #e5e5e5;
  font-weight: 700;
  opacity: .25;
}

.name.compact {
  font-size: .8em;
}
</style>