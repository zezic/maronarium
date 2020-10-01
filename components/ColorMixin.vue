<script>
import chroma from 'chroma-js'
import nearestColor from 'nearest-color'
import namedColors from 'color-name-list'

const colors = namedColors.reduce((o, { name, hex }) => Object.assign(o, { [name]: hex }), {})
const nearest = nearestColor.from(colors)

export default {
  props: {
    rgb: {
      type: Array,
      required: true
    }
  },
  computed: {
    hexLabel () {
      const paddedHex = this.rgb.map(int => {
        return int.toString(16).toUpperCase().padStart(2, '0')
      }).join('')
      return paddedHex
    },
    postfix () {
      return '0'.repeat(6 - this.hexLabel.length)
    },
    style () {
      if (this.rgb.length === 3) {
        const chromaInstance = chroma(this.rgb.slice())
        const hexString = chromaInstance.hex()
        const luminance = chromaInstance.luminance()
        return {
          backgroundColor: hexString,
          color: chromaInstance.set('hsl.l', chromaInstance.get('hsl.l') + (luminance < 0.5 ? 0.4 : -0.4)).hex()
        }
      }
      return {}
    },
    name () {
      if (this.rgb.length === 3) {
        const chromaInstance = chroma(this.rgb.slice())
        const hexString = chromaInstance.hex()
        // const named = namedColors.find(color => color.hex === hexString)
        const named = nearest(hexString)
        return named.name
      }
      return null
    },
    compact () {
      return this.name && this.name.length > 12
    }
  }
}
</script>