<template>
  <div class="tracks">
    <Track
      class="track"
      v-for="(track, i) in tracks"
      :cover="track.cover"
      :artist="track.artist"
      :name="track.name"
      :active="i === active"
      @click="$emit('trackClick', i)"
    />
  </div>
</template>

<script>
import Track from "./Track";

export default {
  components: {
    Track,
  },

  props: {
    tracks: { type: Array, default: () => [] },
    active: { type: Number, default: 0 },
  },
};
</script>

<style lang="sass" scoped>
@import "@/styles/_variables"

.tracks
  padding: $indent-medium

.track
  margin: 0 0 $indent-medium 0
</style>
