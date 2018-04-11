<template lang="html">
  <div class="vue-dataPool" @scroll="handleScroll">
    <ul :style="{paddingTop: scrollElementPaddingTop +'px',paddingBottom: scrollElementPaddingBottom +'px'}">
      <li v-for="(item,index) in displayedData" :key="index">{{item.title}}</li>
    </ul>
    <div class="load-more-gif">loading...</div>
  </div>
</template>

<script>

  let COUNT = 1
  export default {
    name: 'MyInfiniteLoading',
    props: {
      // 数据池 存放所有可能被展示的数据
      dataPool: {
        type: Array,
        required: true,
        default: function () {
          return []
        },
        twoWays: true
      },
      // 每个item的高度
      itemHeight: {
        type: Number,
        default: 44
      },
      // 标识是否能继续滚动
      canScroll: {
        type: Boolean,
        default: true
      },
      // todo ? 作用是什么
      dispatchData: {
        type: Function
      }
    },
    data () {
      return {
        // 上一次滚动的距离
        lastScrollTop: undefined,
        // 预加载的距离
        distance: 44,
        // 屏幕上面未渲染的元素的高度和
        scrollElementPaddingTop: 0,
        // 屏幕下面未渲染的元素的高度和
        scrollElementPaddingBottom: 0,
        // 标识是否能加载更多
        canLoadmore: true,
        // 当前渲染的数据
        displayedData: [],
        // 当前屏幕中显示的第一个item的“次序”
        // 为了在控制面板中显示控制信息而计算的量
        // todo 这个名字起得不好 改名
        displayCount: 0
      }
    },
    mounted () {
      this.initData()
      this.handleScroll()
    },
    methods: {
      initData () {
        // init all data
        // 窗口中显示的item数，向上取整
        this.itemNumsInWindow = Math.ceil(this.$el.offsetHeight / this.itemHeight)
        //  窗口上面未显示的item数
        this.itemNumsAboveWindow = this.itemNumsInWindow * 2
        //  窗口下面未显示的item数
        this.itemNumsBelowWindow = this.itemNumsInWindow
        // 最大高度,当两次相邻滚动之差绝对值>=edgeDistanceForResetDisplayedData,则重新组织数据
        this.edgeDistanceForResetDisplayedData = this.itemNumsInWindow * this.itemHeight
      },
      handleScroll () {
        // 缓存当前的滚动高度
        let currentScrollTop = this.$el.scrollTop
        // 缓存内部滚动元素即ul的高度
        let scrollElmHeight = this.$el.querySelectorAll('ul')[0].offsetHeight
        // 缓存容器高度
        let containerHeight = this.$el.offsetHeight

        // 计算当前屏幕显示的item数目，用于展示数据。不是实现功能必须的量
        this.displayCount = Math.round(currentScrollTop / this.itemHeight)

        // if the maximum height is exceeded, reset the displayedData
        if (this.lastScrollTop === undefined || Math.abs(currentScrollTop - this.lastScrollTop) > this.edgeDistanceForResetDisplayedData) {
          // 更新lastScrollTop
          this.lastScrollTop = currentScrollTop
        } else {
          if (this.lastItemIndexInDisplayedData === this.dataPool.length && scrollElmHeight - currentScrollTop - containerHeight < this.distance) {
            this.canScroll && this.loadMore(this.firstItemIndexInDisplayedData, this.lastItemIndexInDisplayedData)
          }
          return
        }

        // get firstItemIndexInDisplayedData and lastItemIndexInDisplayedData count
        let _firstItemIndexInDisplayedData = parseInt(currentScrollTop / this.itemHeight) - this.itemNumsAboveWindow
        if (_firstItemIndexInDisplayedData < 0) {
          _firstItemIndexInDisplayedData = 0
        }
        let _lastItemIndexInDisplayedData = _firstItemIndexInDisplayedData + this.itemNumsAboveWindow + this.itemNumsBelowWindow + this.itemNumsInWindow
        if (_lastItemIndexInDisplayedData > this.dataPool.length) {
          _lastItemIndexInDisplayedData = this.dataPool.length
        }
        this.firstItemIndexInDisplayedData = _firstItemIndexInDisplayedData
        this.lastItemIndexInDisplayedData = _lastItemIndexInDisplayedData

        // set top height and bottom height
        this.scrollElementPaddingTop = _firstItemIndexInDisplayedData * this.itemHeight
        this.scrollElementPaddingBottom = (this.dataPool.length - _lastItemIndexInDisplayedData) * this.itemHeight

        // dispatch data
        if (typeof this.dispatchData === 'function') {
          this.dispatchData(this)
        }

        // 重新填充要渲染的数据
        this.resetDisplayedData(_firstItemIndexInDisplayedData, _lastItemIndexInDisplayedData)

        // 填充数据后检查是否要加载新数据
        this.$nextTick(() => {
          let currentScrollTop = this.$el.scrollTop
          let scrollElmHeight = this.$el.querySelectorAll('ul')[0].offsetHeight
          let containerHeight = this.$el.offsetHeight

          if (_lastItemIndexInDisplayedData === this.dataPool.length && scrollElmHeight - currentScrollTop - containerHeight < 0) {
            this.canScroll && this.loadMore(this.firstItemIndexInDisplayedData, this.lastItemIndexInDisplayedData)
          }
        })
      },
      loadMore (firstItemIndexInDisplayedData, lastItemIndexInDisplayedData) {
        if (!this.canLoadmore) return
        this.canLoadmore = false
        // fetch mock 模拟异步获取数据的操作
        setTimeout(() => {
          for (var i = 0; i < 200; i++) {
            this.dataPool.push({
              title: 'item ' + COUNT++
            })
          }
          // _firstItemIndexInDisplayedData、 _lastItemIndexInDisplayedData, todo 为了处理数据不能填满屏幕的情况？
          let _firstItemIndexInDisplayedData = firstItemIndexInDisplayedData
          let _lastItemIndexInDisplayedData = lastItemIndexInDisplayedData + this.itemNumsBelowWindow
          this.resetDisplayedData(_firstItemIndexInDisplayedData, _lastItemIndexInDisplayedData)
          this.scrollElementPaddingBottom = (this.dataPool.length - _lastItemIndexInDisplayedData) * this.itemHeight
          this.handleScroll()

          this.canLoadmore = true
        }, 2000)
      },
      resetDisplayedData (firstItemIndexInDisplayedData, lastItemIndexInDisplayedData) {
        // reset displayedData
        this.displayedData = []
        for (let i = firstItemIndexInDisplayedData; i < lastItemIndexInDisplayedData; i++) {
          this.displayedData.push(this.dataPool[i])
        }
      }
    },
    components: {}
  }
</script>

<style lang="stylus">
  .vue-dataPool {
    width: 100%;
    height: 100%;
    overflow-y: auto;
    &::scroll-bar {
      width: 0;
    }
    ul {
      margin: 0;
      padding: 0;
      li {
        text-decoration: none;
        height: 44px;
        font-size: 14px;
        line-height: 3;
        text-align: left;
        padding-left: 15px;
        border-bottom: 1px solid #ddd;
        box-sizing: border-box;
        background: #fff;
        &.line-top, &.line-bottom {
          border-bottom: 0;
        }
      }
    }
    .load-more-gif {
      width: 100%;
      height: 44px;
      text-align: center;
      line-height: 44px;
      background: #fff;
      border-top: none;
      color: #48B884;
    }
  }
</style>
