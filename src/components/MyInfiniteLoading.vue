<template lang="html">
  <div class="vue-list" @scroll="handleScroll">
    <ul :style="{paddingTop: lineTopHeight +'px',paddingBottom: lineBottomHeight +'px'}">
      <li v-for="(item,index) in previewList" :key="index">{{item.title}}</li>
    </ul>
    <div class="load-more-gif">loading...</div>
  </div>
</template>

<script>

  let COUNT = 1
  export default {
    name: 'vue-list',
    props: {
      list: {
        type: Array,
        required: true,
        default: function () {
          return []
        },
        twoWays: true
      },
      height: {
        type: Number,
        default: 44
      },
      canScroll: {
        type: Boolean,
        default: true
      },
      // todo ?
      dispatchData: {
        type: Function
      }
    },
    data () {
      return {
        // 上一次滚动的距离
        lastScrollTop: null,
        distance: 44,
        // 屏幕上面未渲染的元素的高度和
        lineTopHeight: 0,
        // 屏幕下面未渲染的元素的高度和
        lineBottomHeight: 0,
        canLoadmore: true,
        // 当前渲染的数据
        previewList: [],
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
        this._rowsInWindow = Math.ceil(this.$el.offsetHeight / this.height)
        //  窗口上面未显示的item数
        this._above = this._rowsInWindow * 2
        //  窗口下面未显示的item数
        this._below = this._rowsInWindow
        // 最大高度,当两次相邻滚动之差绝对值>=_max,则重新组织数据
        this._max = this._rowsInWindow * this.height
      },
      handleScroll () {
        // 缓存当前的滚动高度
        let _scrollTop = this.$el.scrollTop
        // 缓存内部滚动元素即ul的高度
        let _height = this.$el.querySelectorAll('ul')[0].offsetHeight
        // 缓存容器高度
        let _contentHeight = this.$el.offsetHeight

        // Counts the number of rows on the current screen
        // 计算当前屏幕显示的item数目，用于展示数据。不是实现功能必须的量
        if (_scrollTop / this.height - Math.floor(_scrollTop / this.height) > 0.5) {
          this.displayCount = Math.ceil(_scrollTop / this.height)
        } else {
          this.displayCount = Math.floor(_scrollTop / this.height)
        }

        // if the maximum height is exceeded, reset the previewList
        if (this.lastScrollTop === null || Math.abs(_scrollTop - this.lastScrollTop) > this._max) {
          this.lastScrollTop = _scrollTop
        } else {
          if (this.to === this.list.length && _height - _scrollTop - _contentHeight < this.distance) {
            this.canScroll && this.loadMore(this.from, this.to)
          }
          return
        }

        // get from and to count
        let _from = parseInt(_scrollTop / this.height) - this._above
        if (_from < 0) {
          _from = 0
        }
        let _to = _from + this._above + this._below + this._rowsInWindow
        if (_to > this.list.length) {
          _to = this.list.length
        }
        this.from = _from
        this.to = _to

        // set top height and bottom height
        this.lineTopHeight = _from * this.height
        this.lineBottomHeight = (this.list.length - _to) * this.height

        // dispatch data
        if (typeof this.dispatchData === 'function') {
          this.dispatchData(this)
        }

        // 重新填充要渲染的数据
        this.resetPreviewList(_from, _to)

        // 填充数据后检查是否要加载新数据
        this.$nextTick(() => {
          let _scrollTop = this.$el.scrollTop
          let _height = this.$el.querySelectorAll('ul')[0].offsetHeight
          let _contentHeight = this.$el.offsetHeight

          if (_to === this.list.length && _height - _scrollTop - _contentHeight < 0) {
            this.canScroll && this.loadMore(this.from, this.to)
          }
        })
      },
      loadMore (from, to) {
        if (!this.canLoadmore) return
        this.canLoadmore = false
        // fetch mock 模拟异步获取数据的操作
        setTimeout(() => {
          for (var i = 0; i < 200; i++) {
            this.list.push({
              title: 'item ' + COUNT++
            })
          }
          // _from、 _to, todo 为了处理数据不能填满屏幕的情况？
          let _from = from
          let _to = to + this._below
          this.resetPreviewList(_from, _to)
          this.lineBottomHeight = (this.list.length - _to) * this.height
          this.handleScroll()

          this.canLoadmore = true
        }, 2000)
      },
      resetPreviewList (from, to) {
        // reset previewList
        this.previewList = []
        for (; from < to; from++) {
          this.previewList.push(this.list[from])
        }
      }
    },
    components: {}
  }
</script>

<style lang="stylus">
  .vue-list {
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
