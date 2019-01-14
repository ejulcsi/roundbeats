<template>
  <div id="app">
    <circle-canvas></circle-canvas>
  </div>
</template>

<script>
import circleCanvas from './components/circle.vue'

export default {
  name: 'app',
  components: {
    circleCanvas
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
}
</style>
