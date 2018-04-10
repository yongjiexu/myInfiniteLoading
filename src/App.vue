<template>
  <div id="app">
    <MyInfiniteLoading></MyInfiniteLoading>
  </div>
</template>

<script>
  import MyInfiniteLoading from './components/MyInfiniteLoading'

  export default {
    name: 'App',
    data () {
      return {
        listData: []
      }
    },
    components: {
      MyInfiniteLoading
    }
  }
</script>

<style>

</style>
