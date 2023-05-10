<template>
  <div class="map-container">
    <InfoArea :isDisabled="saveIsDisabled" :addEvent="addPlot" :saveEvent="savePlot"/>
    <div ref="maps" class="map"></div>
  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl'
import axios from 'axios'
import { v4 as uuidv4 } from 'uuid';
import InfoArea from '@/components/info.vue'

export default {
  name: 'PlotsMap',
  components:{
    InfoArea
  },
  data () {
    return {
      maps: null,
      plots: [],
      newPlot:{
        plot_id: '',
        layer: 160,
        points: []
      },
      saveIsDisabled: true,
      addingPlot: false,
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
    this.maps.on('click', (event) => { 
      if(this.addingPlot){
        this.addPoint(event)
      }
    });
  },
  methods: {
    addPlots() {
      axios.get('http://localhost:1337/api/plots', {
      headers: {
        Authorization: 'Bearer e415b0c58749f47b0a303aad4b30b13ed53f39b53db271e6d50f1bb8ef61ba8f29e8d92346df6eae7eda852c2bd9e3c895451913e67439bd2101052fa7348466bd775e54747a273b84222bc23137000bf29f823530f54fece46b4af9d18ad05d1aa9d3b763e392e8970c4be563f0483a2af68a73e1175c6dbb920b92827e3f05'
      }
      })
      .then(response => {
        this.plots = response.data.data
        response.data.data.forEach(plot => {
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
      })
      .catch(error => {
        console.log(error)
      })
    },
    addPlot(){
      this.addingPlot = true
    },
    addPoint(event){
      const el = document.createElement('div');
      el.className = 'marker';
      el.style.width = "10px";
      el.style.height = "10px";
      el.style.borderRadius = "50px";
      el.style.backgroundColor = "#DF1717";
      el.style.position = "absolute";
      el.style.top = "0";
      el.style.left = "0";
      new mapboxgl.Marker({ element: el }).setLngLat([event.lngLat.lng, event.lngLat.lat]).addTo(this.maps);
      const point = {longitude: event.lngLat.lng, latitude: event.lngLat.lat};
      this.newPlot.points.push(point); 
      if (this.newPlot.points.length >= 3) {
        this.saveIsDisabled = false
      }
    },
    savePlot() {
    axios.get(`https://api.mapbox.com/v4/mapbox.mapbox-terrain-v2/tilequery/${this.newPlot.points[0].longitude},${this.newPlot.points[0].latitude}.json?layers=contour&access_token=pk.eyJ1IjoibGl6YWxlaWtvMyIsImEiOiJjbGhmN2JycDMxc3gxM2xudWZhOWIxbHVpIn0.oLNof4WsUYpXqqZka0qpxg`)
    .then(response => {
      const layer = response.data.features[0].properties.ele;
      this.newPlot.layer = layer
    }).catch(error => console.log(error));
    const newPlot = {plot_id: uuidv4(), points: this.newPlot.points};
    this.plots.push(newPlot);
    this.newPlot = {plot_id: '', layer: 0, points: []};
    axios.post(' http://localhost:1337/api/plots', { data: newPlot}, {
      headers: {
        Authorization: 'Bearer e415b0c58749f47b0a303aad4b30b13ed53f39b53db271e6d50f1bb8ef61ba8f29e8d92346df6eae7eda852c2bd9e3c895451913e67439bd2101052fa7348466bd775e54747a273b84222bc23137000bf29f823530f54fece46b4af9d18ad05d1aa9d3b763e392e8970c4be563f0483a2af68a73e1175c6dbb920b92827e3f05'
      }
    })
      .then((response) => {
        console.log(response.data);
        alert('Saved!');
        this.addingPlot = false
      })
      .catch((error) => {
        console.log(error);
      });
      this.addPlots()
    }
  }
}
</script>

<style lang="sass" scoped>
.map-container 
  width: 100%
  height: 100vh
  position: relative

.map 
  width: 100%
  height: 100%

</style>