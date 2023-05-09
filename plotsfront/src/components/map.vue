<template>
  <div class="map-container">
    <div ref="maps" class="map"></div>
  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl'

export default {
  name: 'PlotsMap',
  data () {
    return {
      maps: null,
      // mapsIsInitialized: false
    }
  },
  mounted () {
    mapboxgl.accessToken = 'pk.eyJ1IjoibGl6YWxlaWtvMyIsImEiOiJjbGhmN2JycDMxc3gxM2xudWZhOWIxbHVpIn0.oLNof4WsUYpXqqZka0qpxg'
    this.maps = new mapboxgl.Map({
      container: this.$refs.maps,
      style: 'mapbox://styles/mapbox/streets-v12',
      center: [ 27.561831, 53.902284],
      zoom: 6 
    })
    // this.maps.on('load', () => {
    // //   this.isMapsInitialized = true
    // })
  },
  
}
</script>

<style scoped>
.map-container {
  width: 100%;
  height: 100vh;
}
.map {
  width: 100%;
  height: 100%;
}
</style>