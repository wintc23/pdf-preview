<template>
  <div class="container">
    <div
      v-for="page in pages"
      :key="page"
      ref="pageContainer"
      :style="{
        border: showPageLine ? '1px solid #DFDFDF' : '',
        height: pageHeight + 'px'
      }">
      <canvas v-if="renderList.includes(page)"></canvas>
    </div>
  </div>
</template>

<script>
import pdfJS from 'pdfjs-dist'

export default {
  props: {
    url: {
      type: String,
      required: true
    },
    manualRender: {
      type: Boolean,
      default: false
    },
    renderPages: { // 前后各预渲染多少页
      type: Number,
      default: 5
    },
    showPageLine: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      pdfDoc: null,
      pages: 0,
      currentPage: 0,
      pageHeight: 0,
      renderList: [], // 渲染的页码数组
      timerIndex: 0
    }
  },
  watch: {
    url () {
      this.getPDFFile()
    }
  },
  created () {
    this.getPDFFile()
    if (!this.manualRender) {
      document.addEventListener('scroll', this.documentScroll)
    }
  },
  beforeDestroy () {
    document.removeEventListener('scroll', this.documentScroll)
    this.timerIndex && clearTimeout(this.timerIndex)
  },
  methods: {
    getPDFFile () {
      if (!this.url) return
      pdfJS.getDocument(this.url).then(pdf => {
        this.pdfDoc = pdf
        this.pages = this.pdfDoc.numPages
        this.$nextTick(() => {
          this.pages && this.scrollToPage(1)
        })
      })
    },
    scrollToPage (pageNo) {
      if (this.currentPage === pageNo) return
      this.currentPage = pageNo
      let list = []
      for (let page = pageNo - this.renderPages; page <= pageNo + this.renderPages; page++) {
        list.push(page)
      }
      for (let idx = list.length; idx >= 0; idx--) {
        if (list[idx] > this.pages || list[idx] < 1) list.splice(idx, 1)
      }
      this.$nextTick(() => {
        this.renderList = list
        this.renderList.forEach(page => {
          this.renderPage(page)
        })
      })
    },
    renderPage (pageNo) {
      this.pdfDoc.getPage(pageNo).then(page => {
        let containerNode = this.$refs.pageContainer[pageNo - 1]
        if (!containerNode) return
        let canvas = containerNode.querySelector('canvas')
        if (!canvas) return
        let ctx = canvas.getContext('2d')
        let dpr = window.devicePixelRatio || 1
        let bsr = ctx.webkitBackingStorePixelRatio || ctx.mozBackingStorePixelRatio || ctx.msBackingStorePixelRatio || ctx.oBackingStorePixelRatio || ctx.backingStorePixelRatio || 1
        let ratio = dpr / bsr
        let viewport = page.getViewport(screen.availWidth / page.getViewport(.5).width)
        canvas.width = viewport.width * ratio
        canvas.height = viewport.height * ratio
        canvas.style.width = viewport.width + 'px'
        canvas.style.height = viewport.height + 'px'
        this.pageHeight = viewport.height
        ctx.setTransform(ratio, 0, 0, ratio, 0, 0)
        let renderCtx = {
          canvasContext: ctx,
          viewport
        }
        page.render(renderCtx)
      })
    },
    documentScroll () {
      this.checkRender()
    },
    checkRender () {
      this.timerIndex && clearTimeout(this.timerIndex)
      this.timerIndex = setTimeout(this.renderCurrent, 100)
    },
    renderCurrent () {
      this.$nextTick(() => {
        if (this.pages <= 0) return
        if (!this.pageHeight) return
        if (!this.$refs.pageContainer[0]) return
        let rect = this.$refs.pageContainer[0].getBoundingClientRect()
        if (rect.top >= 0) {
          this.scrollToPage(1)
        } else {
          let page = Math.floor(-rect.top / this.pageHeight) + 1
          if (page > this.pages) page = this.pages
          this.scrollToPage(page)
        }
      })
    }
  }
}
</script>
