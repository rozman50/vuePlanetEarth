<template>
  <div id="app">
      <PlanetEarth/>
      <!-- <Terrain/> -->
  </div>
</template>

<script>
import PlanetEarth from './components/PlanetEarth.vue'
// import Terrain from './components/Terrain.vue'

export default {
  name: 'App',
  components: {
    PlanetEarth
    // Terrain
  }
}
</script>
