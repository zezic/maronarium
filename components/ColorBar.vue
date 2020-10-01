<template>
<div class='color' :style="style">
  <div class="hex-label">
    <span class="prefix">0x</span><span class="hex-string">{{ hexLabel }}</span><span class="postfix">{{ postfix }}</span>
  </div>
  <div class='name' :class='{ compact }' v-if='name'>{{ name }}</div>
</div>
</template>

<script>
import ColorMixin from '~/components/ColorMixin'

export default {
  mixins: [ColorMixin]
}
</script>

<style scoped>
.color {
  display: flex;
  height: 12vw;
  border-radius: 6vw;
  align-items: center;
  justify-content: space-between;
  padding: 0 9vw;
  transition: background-color ease .2s;
  background-color: #2a2a2a;
  color: #e5e5e5;
  font-weight: 700;
  font-size: 4.5vw;
  line-height: 0;
}

.hex-label .prefix,
.hex-label .postfix {
  opacity: .15;
  font-weight: 400;
}

.hex-label > * {
  vertical-align: top;
}

@media screen and (min-width: 640px) {
  .color {
    height: 40px;
    border-radius: 20px;
    font-size: 16px;
    padding: 0 32px;
  }
}

.name {
  text-align: right;
  white-space: normal;
  line-height: 1.1;
  padding-left: 1em;
}

.name.compact {
  font-size: .8em;
}
</style>