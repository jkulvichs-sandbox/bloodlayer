<template>
  <!--  <Icon icon="play-speed" color="red" size="medium"></Icon>-->
  <!--  <Track cover="./albums/Slaughter to Prevail.jpg" :active="true"/>-->
  <TrackList :tracks="tracks" :active="active" @trackClick="onTrackClick"/>
</template>

<script>
import Icon from './components/Icon.vue'
import TrackList from './components/TrackList'

export default {
  components: {
    Icon,
    TrackList,
  },

  data() {
    return {
      active: -1,
      tracks: new Array(10).fill({}).map(() => {
        const tracks = [
          {
            cover: './albums/Slaughter to Prevail.jpg',
            artist: 'Slaughter to Prevail',
            name: 'Zavali Ebalo'
          },
          {
            cover: './albums/Slaughter to Prevail.jpg',
            artist: 'Slaughter to Prevail',
            name: 'Intro'
          },
          {
            cover: './albums/Slaughter to Prevail.jpg',
            artist: 'Slaughter to Prevail',
            name: 'Bratva'
          },
        ]
        return tracks[Math.trunc(tracks.length * Math.random())]
      })
    }
  },

  methods: {
    onTrackClick(i) {
      this.active = i
    }
  },
}
</script>

<style lang="sass" scoped>

</style>
