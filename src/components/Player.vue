<template>
  <div class="player">
    <div class="trackLine">
      <div class="controls">
        <Icon icon="skip-previous" size="large" />
        <Icon icon="play-speed" size="large" />
        <Icon icon="skip-next" size="large" />
      </div>
    </div>
  </div>
</template>

<script>
import Icon from "@/components/Icon.vue";

export default {
  components: {
    Icon,
  },
};
</script>

<style lang="sass" scoped>
@import "@/styles/_variables"

.player
    position: sticky
    display: flex

    width: 100%
    bottom: 0
    z-index: 99

.controls
  display: flex
  justify-content: space-around
  align-items: center
  height: 100%
  width: 300px
  max-width: 200px
  padding: $indent-medium 0
  background: $color-foreground
  border-radius: 0 10px 10px 0
  box-shadow: 2px 0 5px rgba(0, 0, 0, .1)

.trackLine
  width: 100%
  background: rgba(#FFFFFF, 0.8)
  backdrop-filter: blur(10px) //invert(1)
  height: 100%
  overflow: hidden
</style>
