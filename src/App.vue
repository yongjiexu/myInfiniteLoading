<template>
  <div id="app">
    <header>
      <h1 class="title">myInfiniteLoading</h1>
      <p>A solution to build an infinite load more list component.</p>
      <p><a class="button" href="https://github.com/yongjiexu/myInfiniteLoading">Code on Github</a> / <a class="button" href="https://github.com/yongjiexu">My
        profile</a></p>
    </header>
    <div class="content">
      <div class="preview">
        <div class="preview-content">
          <MyInfiniteLoading :dataPool.sync='list' :dispatchData="setData"></MyInfiniteLoading>
        </div>
      </div>
      <div class="setting">
        <h3>Scroll info</h3>
        <p>Total items: <span>{{list.length}}</span></p>
        <p>Item height: <span>{{data.itemHeight}}px</span></p>
        <p>Above items: <span>{{data.itmNumsAbvWnd}}</span></p>
        <p>Below items: <span>{{data.itmNumsBlwWnd}}</span></p>
        <p>Rows in window: <span>{{data.displayCount + 1}}-{{(data.displayCount + data.itmNumsInWnd) > list.length ? list.length: (data.displayCount + data.itmNumsInWnd)}}</span>
        </p>
        <p><span>{{data.itmNumsInWnd * 4}}</span> items from <span>{{data.firstItmIdxDisplayedData}}</span> to <span>{{data.lastItmIdxDisplayedData}}</span></p>
        <p>Top height: <span>{{data.scrollElmPaddingTop}}px</span></p>
        <p>Bottom Height: <span>{{data.scrollElmPaddingBotm}}px</span></p>
        <p>Will load more items: <span>{{!data.canLoadMore}}</span></p>
        <br>
        <p>
          <span>You can open the developer tools and observe the changes in the elements.</span>
        </p>
      </div>
    </div>
  </div>
</template>

<script>
  import MyInfiniteLoading from 'components/MyInfiniteLoading'

  let COUNT = 1

  export default {
    name: 'app',
    data () {
      return {
        list: [],
        data: {}
      }
    },
    created () {
      for (var i = 0; i < 100; i++) {
        this.list.push({
          title: 'item ' + COUNT++
        })
      }
    },
    methods: {
      setData (data) {
        this.data = data
      }
    },
    components: {
      MyInfiniteLoading
    }
  }
</script>

<style lang="stylus">
  html, body {
    margin: 0;
    padding: 0;
  }

  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    header {
      width: 100%;
      padding: 30px 0;
      margin-bottom: 20px;
      .title {
        font-size: 46px;
        line-height: 20px;
        color: #48B884;
      }
      p {
        color: #7F8C8D;
        .button {
          color: #48B884;
        }
      }
    }
    .content {
      display: flex;
      width: 800px;
      height: 650px;
      margin: 0 auto;
      & > div {
        flex: 1;
        height: 100%;
      }
      .preview {
        background: url("~assets/iphone.png") center center no-repeat
        background-size: 324px 642px;
        position: relative;
        .preview-content {
          width: 256px;
          height: 464px;
          position: absolute;
          left: 72px;
          top: 91px;
          background: #fff;
          overflow-y: scroll;
        }
      }
      .setting {
        span {
          color: #48B884;
        }
      }
    }
  }
</style>
