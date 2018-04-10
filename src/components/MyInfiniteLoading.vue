<template>
  <div id="my-infinite-loading-wrapper">
    <div class="scroll-container" @scroll="handleScroll">
      <ul>
        <li v-for="(item,index) in dataPool" :key="index">item {{item}}</li>
      </ul>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'MyInfiniteLoading',
    data () {
      return {
        // 存放所有可能要展示的数据
        dataPool: [],
        scrollContainer: undefined
      }
    },
    props: {
      distance: {
        type: Number,
        required: false,
        default: 30
      }
    },
    computed: {
      // 存放当前要展示的数据
      displayData: function () {
        return this.dataPool.slice(0, 30)
      }
    },
    created () {
      for (let i = 0; i < 15; i++) {
        this.dataPool.push(i)
      }
    },
    mounted () {
      this.scrollContainer = document.querySelector('.scroll-container')
    },
    methods: {
      // 模拟网络操作，加载更多数据
      loadMore () {
        setTimeout(() => {
          const temp = []
          for (let i = this.dataPool.length + 1; i <= this.dataPool.length + 10; i++) {
            temp.push(i)
          }
          this.dataPool = this.dataPool.concat(temp)
        }, 10)
      },
      /**
       * 处理滚动事件，检测是否应该加载更多
       */
      handleScroll () {
        if (this.checkBottomReached()) {
          this.loadMore()
        }
      },
      checkBottomReached () {
        return this.scrollContainer.scrollTop + this.scrollContainer.offsetHeight + this.distance >= this.scrollContainer.scrollHeight
      }
    }
  }
</script>

<style scoped lang="stylus">
  #my-infinite-loading-wrapper
    width: 100%
    height: 100vh
    .scroll-container
      width: 100%
      height: 700px
      overflow-y scroll
      outline: 1px solid red
      ul
        width: 100%
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
