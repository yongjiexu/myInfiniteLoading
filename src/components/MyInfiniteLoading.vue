<template>
  <div id="container">
    <ul>
      <li v-for="(item,index) in displayData" :key="index">item {{item}}</li>
    </ul>
    <!--<div class="load-more">loading...</div>-->
  </div>
</template>

<script>
  export default {
    name: 'MyInfiniteLoading',
    data () {
      return {
        displayData: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30]
      }
    }
  }
</script>

<style scoped lang="stylus">
  div#container
    width: 100%
    height: 100vh
    overflow-y auto
    ul
      display: block
      width: 100%
      height: 100%
      list-style: none
      li
        text-decoration none
        width: 100%
        height: 44px
        font-size 14px
        line-height 44px
        padding-left 15px
        text-align left
        border-bottom 1px solid #ddd
        background: #fff
    div
      width: 100%
      height: 44px
      text-align: center
      line-height: 44px
      background: #fff
      border-top: none
      color: #48B884
</style>
