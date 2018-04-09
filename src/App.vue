<template>
  <div id="app">
    <MyInfiniteLoading></MyInfiniteLoading>
  </div>
</template>

<script>
import MyInfiniteLoading from './components/MyInfiniteLoading'

export default {
  name: 'App',
  components: {
    MyInfiniteLoading
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
}
</style>
