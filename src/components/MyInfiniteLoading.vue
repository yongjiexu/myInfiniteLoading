<template lang="html">
  <div class="my-infinite-loading" @scroll="handleScroll">
    <ul :style="{paddingTop: scrollElmPaddingTop +'px',paddingBottom: scrollElmPaddingBotm +'px'}">
      <li v-for="(item,index) in displayedData" :key="index" v-if="item">{{item.title}}</li>
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
      // 标识是否允许滚动
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
        // 上一次更新displayedData时scrollContainer的scrollTop
        lastScrollTop: undefined,
        // 当前scrollContainer的scrollTop
        currentScrollTop: undefined,
        // 预加载的距离
        distance: 44,
        // 屏幕上面未渲染的元素的高度和
        scrollElmPaddingTop: 0,
        // 屏幕下面未渲染的元素的高度和
        scrollElmPaddingBotm: 0,
        // 标识是否能加载更多
        canLoadMore: true,
        // 当前渲染的数据
        displayedData: [],
        // 展示数据的第一项的下标(这个下标是相对dataPool计算的)
        firstItmIdxDisplayedData: undefined,
        // 展示数据的最后一项的下标(这个下标是相对dataPool计算的)
        lastItmIdxDisplayedData: undefined,
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
        // init all data itmNumsInWnd
        // 窗口中显示的item数，向上取整
        this.itmNumsInWnd = Math.ceil(this.$el.offsetHeight / this.itemHeight)
        //  窗口上面未显示的item数
        this.itmNumsAbvWnd = this.itmNumsInWnd * 2
        //  窗口下面未显示的item数
        this.itmNumsBlwWnd = this.itmNumsInWnd
        // 最大高度,当两次相邻滚动之差绝对值>=edgeScrollDistance,则重新组织数据
        this.edgeScrollDistance = this.itmNumsInWnd * this.itemHeight
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

        // 只有上一次scrollTop未设置 或 当前scrollTop-lastScrollTop之差大于临界值时，才更新lastScrollTop，更新displayedData
        if (this.lastScrollTop === undefined || Math.abs(currentScrollTop - this.lastScrollTop) >= this.edgeScrollDistance) {
          // 更新lastScrollTop
          this.lastScrollTop = currentScrollTop
          // 计算新的displayedData的第一项在dataPool中的下标 即_firstItmIdxDisplayedData
          let _firstItmIdxDisplayedData = Math.round(currentScrollTop / this.itemHeight) - this.itmNumsAbvWnd
          if (_firstItmIdxDisplayedData < 0) {
            _firstItmIdxDisplayedData = 0
          }
          // 计算新的displayedData的最后一项在dataPool中的下标 即_lastItmIdxDisplayedData
          let _lastItmIdxDisplayedData = _firstItmIdxDisplayedData + this.itmNumsAbvWnd + this.itmNumsBlwWnd + this.itmNumsInWnd
          if (_lastItmIdxDisplayedData > this.dataPool.length) {
            _lastItmIdxDisplayedData = this.dataPool.length
          }
          // 更新firstItmIdxDisplayedData、lastItmIdxDisplayedData
          this.firstItmIdxDisplayedData = _firstItmIdxDisplayedData
          this.lastItmIdxDisplayedData = _lastItmIdxDisplayedData

          // 更新滚动元素的paddingTop、paddingBottom
          this.scrollElmPaddingTop = _firstItmIdxDisplayedData * this.itemHeight
          this.scrollElmPaddingBotm = (this.dataPool.length - _lastItmIdxDisplayedData) * this.itemHeight

          // dispatch data
          if (typeof this.dispatchData === 'function') {
            this.dispatchData(this)
          }

          // 重新填充要渲染的数据
          this.resetDisplayedData(_firstItmIdxDisplayedData, _lastItmIdxDisplayedData)
        } else {
          // 当我们当前展示的数据是数据池中最后一段数据 且 剩余未展示的内容高度小于预加载距离时，需要重置展示数据。但是数据已经不够了，故从服务器获取更多数据
          // scrollElmHeight - currentScrollTop - containerHeight < 0 这句代码好理解：我们模拟了完全展示了dataPool中数据的情况。
          // 这句话的语义是：剩余未展示的内容高度小于预加载距离
          if (this.lastItmIdxDisplayedData === this.dataPool.length && scrollElmHeight - currentScrollTop - containerHeight < this.distance) {
            this.canScroll && this.loadMore(this.firstItmIdxDisplayedData, this.lastItmIdxDisplayedData)
          }
          // return
        }
      },
      loadMore (firstItmIdxDisplayedData, lastItmIdxDisplayedData) {
        if (!this.canLoadMore) return
        this.canLoadMore = false
        // fetch mock 模拟异步获取数据的操作
        setTimeout(() => {
          for (var i = 0; i < 100; i++) {
            this.dataPool.push({
              title: 'item ' + COUNT++
            })
          }
          // _firstItmIdxDisplayedData、 _lastItmIdxDisplayedData, todo 为了处理数据不能填满屏幕的情况？
          let _firstItmIdxDisplayedData = firstItmIdxDisplayedData
          let _lastItmIdxDisplayedData = lastItmIdxDisplayedData + this.itmNumsBlwWnd
          this.resetDisplayedData(_firstItmIdxDisplayedData, _lastItmIdxDisplayedData)
          this.scrollElmPaddingBotm = (this.dataPool.length - _lastItmIdxDisplayedData) * this.itemHeight
          this.canLoadMore = true
        }, 2000)
      },
      resetDisplayedData (firstItmIdxDisplayedData, lastItmIdxDisplayedData) {
        // reset displayedData
        this.displayedData = []
        for (let i = firstItmIdxDisplayedData; i < lastItmIdxDisplayedData; i++) {
          this.displayedData.push(this.dataPool[i])
        }
      }
    },
    components: {}
  }
</script>

<style lang="stylus">
  .my-infinite-loading {
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
