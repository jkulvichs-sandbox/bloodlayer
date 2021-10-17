<template>
  <svg :width="sizePixels" :height="sizePixels" viewBox="0 0 24 24">
    <path :fill="color" :d="iconPath"/>
  </svg>
</template>

<script>
import {
  mdiPlaySpeed,
  mdiSkipNext,
  mdiSkipPrevious,
} from '@mdi/js'

const paths = {
  'play-speed': mdiPlaySpeed,
  'skip-next': mdiSkipNext,
  'skip-previous': mdiSkipPrevious,
}

const sizes = {
  'x-small': 12,
  'small': 16,
  'medium': 24,
  'large': 36,
  'x-large': 48,
}

export default {
  props: {
    color: {type: String, default: '#000000'},
    size: {type: String, default: 'medium'},
    icon: {type: String, required: true}
  },

  data() {
    return {
      path: '',
    }
  },

  computed: {
    sizePixels() {
      // Три способа проверить, что значение в словаре отсутствует
      // if (sizes[this.size] == null) {}
      // if (!sizes[this.size]) {}
      // if (!(this.size in sizes)) {}
      if (!sizes[this.size]) {
        throw new Error(`Incorrect size, got ${this.size}`)
      }
      return sizes[this.size]
    },
    iconPath() {
      if (!paths[this.icon]) {
        throw new Error(`Incorrect icon name, got ${this.icon}`)
      }
      return paths[this.icon]
    }
  }
}
</script>

<style lang="sass" scoped>
</style>
