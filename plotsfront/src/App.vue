<template>
    <PlotsMap/>
</template>

<script>
import PlotsMap from './components/map.vue'

export default {
  name: 'App',
  components: {
    PlotsMap
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #000;
}
</style>
