<template>
  <div class="map-container">
    <div ref="maps" class="map"></div>
  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl'
import axios from 'axios'

export default {
  name: 'PlotsMap',
  data () {
    return {
      maps: null,
      plots: [],
      newPlot:{
        plot_id: '',
        layer: 0,
        points: []
      }
    }
  },
  async mounted () {
    mapboxgl.accessToken = 'pk.eyJ1IjoibGl6YWxlaWtvMyIsImEiOiJjbGhmN2JycDMxc3gxM2xudWZhOWIxbHVpIn0.oLNof4WsUYpXqqZka0qpxg'
    this.maps = new mapboxgl.Map({
      container: this.$refs.maps,
      style: 'mapbox://styles/mapbox/streets-v12',
      center: [ 27.561831, 53.902284],
      zoom: 6 
    })

    this.maps.on('load', () => {
      this.addPlots()
    })
    axios.get('http://localhost:1337/api/plots', {
      headers: {
        Authorization: 'Bearer e415b0c58749f47b0a303aad4b30b13ed53f39b53db271e6d50f1bb8ef61ba8f29e8d92346df6eae7eda852c2bd9e3c895451913e67439bd2101052fa7348466bd775e54747a273b84222bc23137000bf29f823530f54fece46b4af9d18ad05d1aa9d3b763e392e8970c4be563f0483a2af68a73e1175c6dbb920b92827e3f05'
      }
    })
    .then(response => {
      this.plots = response.data.data
      this.addPlots()
    })
    .catch(error => {
      console.log(error)
    })
  },
  methods: {
    addPlots() {
      this.plots.forEach(plot => {
        const coordinates = plot.attributes.points.map(point => [point.longitude, point.latitude])
        if (!this.maps.getLayer(plot.attributes.plot_id)) { 
        this.maps.addLayer({
          id: plot.attributes.plot_id, 
          type: 'fill',
          source: {
            type: 'geojson',
            data: {
              type: 'Feature',
              properties: {},
              geometry: {
                type: 'Polygon',
                coordinates: [coordinates]
              }
            }
          },
          paint: {
            'fill-color': '#DF1717',
            'fill-opacity': 0.6
          }
        })
      }
      })
    },
  }
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