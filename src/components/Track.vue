<template>
  <div class="track d-flex rounded" :class="{ 'active': active }">
    <img :src="cover" class="cover rounded">
    <div class="info d-flex flex-column">
      <div class="artist font-large" :class="{ active }">{{ artist }}</div>
      <div class="name" :class="{ active }">{{ name }}</div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    cover: {type: String, default: ''},
    artist: {type: String, default: 'Unknown Artist'},
    name: {type: String, default: 'Unnamed'},
    active: {type: Boolean, default: false},
  }
}
</script>

<style lang="sass" scoped>
.track
  padding: 10px 10px 10px 10px
  cursor: pointer
  background: #333333
  transition: .1s ease-out
  &.active
    background: #FAFAFA
    padding: 10px 10px 10px 20px

.cover
  width: 60px
  height: 60px
  background: #EDE734
  object-fit: cover
  margin: 0 10px 0 0

.info
  justify-content: center

  .artist
    margin: 0 0 10px 0
    color: #FFFFFF
    &.active
      color: #212121

  .name
    opacity: .8
    color: #FFFFFF
    &.active
      color: #212121
</style>
