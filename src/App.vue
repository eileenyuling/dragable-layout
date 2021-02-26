<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <ScaleContainer msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import ScaleContainer from './components/ScaleContainer.vue'

export default {
  name: 'App',
  components: {
    ScaleContainer
  }
}
</script>

<style>
body {
  margin: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
}
</style>
