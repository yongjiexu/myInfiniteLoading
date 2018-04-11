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
  /*
   * 频率控制 返回函数连续调用时，fn 执行频率限定为每多少时间执行一次
   * @param fn {function}  需要调用的函数
   * @param delay  {number}    延迟时间，单位毫秒
   * @param immediate  {bool} 给 immediate参数传递false 绑定的函数先执行，而不是delay后执行。
   * @return {function}实际调用函数
   */
  function throttle (fn, delay) {
    var now, lastExec, timer, context, args //eslint-disable-line

    var execute = function () {
      fn.apply(context, args)
      lastExec = now
    }

    return function () {
      context = this
      args = arguments

      now = Date.now()

      if (timer) {
        clearTimeout(timer)
        timer = null
      }

      if (lastExec) {
        var diff = delay - (now - lastExec)
        if (diff < 0) {
          execute()
        } else {
          timer = setTimeout(() => {
            execute()
          }, diff)
        }
      } else {
        execute()
      }
    }
  }

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
        default: 50
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
      this.init()
    },
    methods: {
      /**
       做一些初始化工作
       */
      init () {

      },
      /**
       * 处理滚动事件，检测是否应该加载更多
       */
      _handleScroll () {
        if (this.checkBottomReached()) {
          this.loadMore()
        }
      },
      handleScroll () {
        throttle(this._handleScroll, 100)()
      },
      /**
       * 检查是否滚动到底部
       * @returns {boolean}
       */
      checkBottomReached () {
        return this.scrollContainer.scrollTop + this.scrollContainer.offsetHeight + this.distance >= this.scrollContainer.scrollHeight
      },
      // 模拟网络操作，加载更多数据
      loadMore () {
        setTimeout(() => {
          const temp = []
          for (let i = this.dataPool.length + 1; i <= this.dataPool.length + 10; i++) {
            temp.push(i)
          }
          this.dataPool = this.dataPool.concat(temp)
        }, 10)
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
