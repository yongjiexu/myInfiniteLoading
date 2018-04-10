<template>
  <div id="app">
    <MyInfiniteLoading :dataPool="listData" :loadMoreCallback="loadMore"></MyInfiniteLoading>
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
    created () {
      for (let i = 0; i < 50; i++) {
        this.listData.push (`item ${++i}`)
      }
    },
    method:{
      loadMore(){
        setTimeout(() => {
          const temp = [];
          for (let i = this.listData.length + 1; i <= this.listData.length + 20; i++) {
            temp.push(i);
          }
          this.listData = this.listData.concat(temp);
        }, 1000);
      }
    },
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
