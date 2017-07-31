<template>
  <div class="one-paging">
    <span class="one-paging-back" @click="back">&lt;</span>
    <span class="one-paging-startTag" v-if="showStartTag" :class="{'one-paging-selected': 1 === selected}" @click="selectedPageNumber(1)">{{1}}</span>
    <span class="one-paging-fastBack" v-if="showFastBack" @click="fastBack">...</span>
    <ul class="one-paging-list">
      <!-- 页码渲染区  -->
      <li v-for=" i in renderCount" class="one-paging-list-item" :class="{'one-paging-selected': renderStart+i-1 === selected}" :key="i.id" @click="selectedPageNumber(renderStart+i-1)">
        {{renderStart+i-1}}
      </li>
    </ul>
    <span class="one-paging-fastNext" v-if="showFastNext" @click="fastNext">...</span>
    <span class="one-paging-endTag" v-if="showEndTag" :class="{'one-paging-selected': pageCount === selected}" @click="selectedPageNumber(pageCount)">{{pageCount}}</span>
    <span class="one-paging-next" @click="next">&gt;</span>
  </div>
</template>

<script>
export default {
  props: {
    pageCount: {
      default: 0
    },
    showCount: {
      default: 5
    }
  },
  data () {
    return {
      selected: '', // 当前选中
      showFastBack: false, // 显示页码快退
      showFastNext: false, // 显示页码快进
      showStartTag: false, // 显示总页码开端
      showEndTag: false, // 显示总页码
      renderStart: '' // 页码渲染起始
    }
  },
  mounted () {
    this.selected = 1 // 默认选中
    this.renderStart = 1 // 页码渲染起始
  },
  methods: {
    back () {
      if (this.selected <= 1) alert('已经到首页了') // 是否可以后退
      else this.selected--
    },
    next () {
      if (this.selected >= this.pageCount) alert('已经是最后一页了') // 是否可以快进
      else this.selected++
    },
    fastBack () {
      // 先改变renderStart 避免先改动selected时会导致fast进行两次
      if (this.renderStart - this.showCount < 1) {
        this.renderStart = 1
      } else {
        this.renderStart -= this.showCount
      }

      // 改变selected
      this.selected = this.selected - this.showCount > 0 ? this.selected - this.showCount : 1
    },
    fastNext () {
      if (this.renderStart + this.showCount > this.pageCount) {
        this.renderStart = this.pageCount - this.showCount
      } else {
        this.renderStart += this.showCount
      }
      this.selected = this.selected + this.showCount <= this.pageCount ? this.selected + this.showCount : this.pageCount
    },
    selectedPageNumber (index) {
      this.selected = index
    }
  },
  computed: {
    renderCount () { // 页码实际渲染数量
      // 计算renderCount
      if (this.renderStart + this.showCount - 1 > this.pageCount) {
        return this.pageCount - this.renderStart + 1
      } else {
        return this.showCount
      }
    }
  },
  watch: {
    selected (val, oldVal) { // next 和 fast会控制selected的值不会越界,改监听主要用于改变renderStart
      // 判断是否超出当前渲染区域,超出改变渲染区
      if (val < this.renderStart) {
        this.renderStart = this.renderStart - this.showCount > 0 ? this.renderStart - this.showCount : 1
      } else if (val === this.pageCount && !(this.renderStart + this.showCount === this.pageCount)) { // 当选中最后一页到时候，且不是(1234..5)会被渲染为(12345 )的情况
        if (this.pageCount - this.showCount >= 0) this.renderStart = this.pageCount - this.showCount + 1
      } else if (val > this.renderStart + this.showCount - 1) { // 如果selected大于渲染区域，且不是(1234..5)会渲染成(12345)的情况
        this.renderStart = val // 改变renderStart
      }
      this.$emit('selectChanged', val)// 给父级抛出的事件,抛出被选中的页面的索引
    },
    renderStart (val, oldVal) {
      // 判断是否显示快进按钮和startTag
      if (this.renderStart === 1) {
        this.showStartTag = false
        this.showFastBack = false
      } else if (this.renderStart === 2) {
        this.showStartTag = true
        this.showFastBack = false
      } else if (this.renderStart > 2) {
        this.showStartTag = true
        this.showFastBack = true
      }

      // 判断是否显示快退按钮和endTag
      if (this.renderStart + this.showCount - 1 >= this.pageCount) {
        this.showEndTag = false
        this.showFastNext = false
      } else if (this.renderStart + this.showCount === this.pageCount) {
        this.showEndTag = true
        this.showFastNext = false
      } else if (this.renderStart + this.showCount < this.pageCount) {
        this.showEndTag = true
        this.showFastNext = true
      }
    }
  }
}
</script>

<style lang='scss'>
.one {
  &-paging {
    user-select: none;
    cursor: default;
    &-back,&-next,&-fastBack,&-fastNext,&-startTag,&-endTag,&-list,&-list-item{
      display: inline-block;
      cursor: pointer;
    }
    &-selected {
      color: red
    }
  }
}
</style>
