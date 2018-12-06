<template>
  <div id="app">
    <converter />
  </div>
</template>

<script>
import Converter from './components/Converter.vue'

export default {
  name: 'app',
  components: {
    Converter
  }
}
</script>

<style>
  html, body {
    margin: 0;
    padding: 0;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
  }
</style>
