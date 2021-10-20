<template>
  <div class="track " :class="{ active: active }">
    <img :src="cover" class="cover" />
    <div class="info">
      <div class="artist" :class="{ active }">{{ artist }}</div>
      <div class="name" :class="{ active }">{{ name }}</div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    cover: { type: String, default: "" },
    artist: { type: String, default: "Unknown Artist" },
    name: { type: String, default: "Unnamed" },
    active: { type: Boolean, default: false },
  },
};
</script>

<style lang="sass" scoped>
@import "@/styles/_variables"


.track
  display: flex
  align-items: center
  padding: $indent-medium
  cursor: pointer
  background: $color-background
  transition: .1s ease-out
  border-radius: $radius
  &.active
    background: $color-foreground
    padding: 10px 10px 10px 20px

.cover
  width: 40px
  height: 40px
  background: $color-accent
  object-fit: cover
  margin: 0 $indent-medium 0 0
  border-radius: $radius

.info
  justify-content: center

.artist
  margin: 0 0 $indent-small 0
  color: $color-foreground
  &.active
    color: $color-primary

.name
  opacity: .8
  color: $color-foreground
  font-size: $font-medium
  &.active
    color: $color-primary
</style>
