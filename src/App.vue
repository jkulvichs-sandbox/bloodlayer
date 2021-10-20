<template>
  <!--  <Icon icon="play-speed" color="red" size="medium"></Icon>-->
  <!--  <Track cover="./albums/Slaughter to Prevail.jpg" :active="true"/>-->
  <TrackList :tracks="tracks" :active="active" @trackClick="onTrackClick" />
  <Player />
</template>

<script>
import Icon from "./components/Icon.vue";
import TrackList from "./components/TrackList";
import Player from "./components/Player";

export default {
  components: {
    Icon,
    TrackList,
    Player,
  },

  data() {
    return {
      active: -1,
      tracks: new Array(15).fill({}).map(() => {
        const tracks = [
          {
            cover: "./albums/Slaughter to Prevail.jpg",
            artist: "Slaughter to Prevail",
            name: "Zavali Ebalo",
          },
          {
            cover: "./albums/Slaughter to Prevail.jpg",
            artist: "Slaughter to Prevail",
            name: "Intro",
          },
          {
            cover: "./albums/Slaughter to Prevail.jpg",
            artist: "Slaughter to Prevail",
            name: "Bratva",
          },
        ];
        return tracks[Math.trunc(tracks.length * Math.random())];
      }),
    };
  },

  methods: {
    onTrackClick(i) {
      this.active = i;
      navigator.vibrate([10, 10, 5, 5, 3]);
      // navigator.vibrate([100, 100, 50, 50, 25, 25, 12, 12, 6, 6, 3, 3])
    },
  },
};
</script>

<style lang="sass"></style>
