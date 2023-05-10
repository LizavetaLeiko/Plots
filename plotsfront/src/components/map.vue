<template>
  <div class="map-container">
    <InfoArea
      :isDisabled="saveIsDisabled"
      :addEvent="addPlot"
      :saveEvent="savePlot"
    />
    <div
      ref="maps"
      class="map"
      :style="addingPlot ? { cursor: 'pointer' } : { cursor: 'default' }"
    ></div>
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import axios from "axios";
import { v4 as uuidv4 } from "uuid";
import InfoArea from "@/components/info.vue";

export default {
  name: "PlotsMap",
  components: {
    InfoArea,
  },
  data() {
    return {
      maps: null,
      plots: [],
      newPlot: {
        plot_id: "",
        layer: 160,
        points: [],
      },
      saveIsDisabled: true,
      addingPlot: false,
      markers: [],
    };
  },
  async mounted() {
    mapboxgl.accessToken = process.env.VUE_APP_MAPBOX_TOKEN;
    this.maps = new mapboxgl.Map({
      container: this.$refs.maps,
      style: "mapbox://styles/mapbox/streets-v12",
      center: [27.561831, 53.902284],
      zoom: 6,
    });
    this.maps.on("load", () => {
      this.addPlots();
    });
    this.maps.on("click", (event) => {
      if (this.addingPlot) {
        this.addPoint(event);
      }
    });
  },
  methods: {
    addPlots() {
      axios
        .get("http://localhost:1337/api/plots", {
          headers: {
            Authorization: `Bearer ${process.env.VUE_APP_STRAPI_TOKEN}`,
          },
        })
        .then((response) => {
          this.plots = response.data.data;
          response.data.data.forEach((plot) => {
            const coordinates = plot.attributes.points.map((point) => [
              point.longitude,
              point.latitude,
            ]);
            if (!this.maps.getLayer(plot.attributes.plot_id)) {
              this.maps.addLayer({
                id: plot.attributes.plot_id,
                type: "fill",
                source: {
                  type: "geojson",
                  data: {
                    type: "Feature",
                    properties: {},
                    geometry: {
                      type: "Polygon",
                      coordinates: [coordinates],
                    },
                  },
                },
                paint: {
                  "fill-color": "#DF1717",
                  "fill-opacity": 0.6,
                },
              });
            }
          });
        })
        .catch((error) => {
          console.log(error);
        });
    },
    addPlot() {
      this.addingPlot = true;
    },
    addPoint(event) {
      const el = document.createElement("div");
      el.className = "marker2";
      el.style.width = "10px";
      el.style.height = "10px";
      el.style.borderRadius = "50px";
      el.style.backgroundColor = "#DF1717";
      el.style.position = "absolute";
      el.style.top = "0";
      el.style.left = "0";
      const marker = new mapboxgl.Marker({ element: el })
        .setLngLat([event.lngLat.lng, event.lngLat.lat])
        .addTo(this.maps);
      this.markers.push(marker);
      const point = { longitude: event.lngLat.lng, latitude: event.lngLat.lat };
      this.newPlot.points.push(point);
      if (this.newPlot.points.length >= 3) {
        this.saveIsDisabled = false;
      }
    },
    removeMarkers() {
      this.markers.forEach((marker) => {
        marker.remove();
      });
      this.markers = [];
    },
    savePlot() {
      axios
        .get(
          `https://api.mapbox.com/v4/mapbox.mapbox-terrain-v2/tilequery/${this.newPlot.points[0].longitude},${this.newPlot.points[0].latitude}.json?layers=contour&access_token=${process.env.VUE_APP_MAPBOX_TOKEN}`
        )
        .then((response) => {
          const layer = response.data.features[0].properties.ele;
          this.newPlot.layer = layer;
        })
        .catch((error) => console.log(error));
      const newPlot = {
        plot_id: uuidv4(),
        layer: this.newPlot.layer,
        points: this.newPlot.points,
      };
      this.plots.push(newPlot);
      this.newPlot = { plot_id: "", layer: 0, points: [] };

      axios
        .post(
          " http://localhost:1337/api/plots",
          { data: newPlot },
          {
            headers: {
              Authorization: `Bearer ${process.env.VUE_APP_STRAPI_TOKEN}`,
            },
          }
        )
        .then((response) => {
          console.log(response.data);
          alert("Saved!");
          this.addingPlot = false;
        })
        .catch((error) => {
          console.log(error);
        });
      this.addPlots();
      this.removeMarkers();
      this.saveIsDisabled = true
    },
  },
};
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
