<template>
  <div>
    <div style="margin-bottom:10px">
      <button @click="goPage()">跳转到第{{pageNumber}}页</button>
      <button @click="goSearch()"
              style="margin-left:5px">从第{{pageNumber}}页开始搜索“{{searchText}}”</button>
      <button @click="openSearch()"
              style="margin-left:5px">弹出搜索框搜索“{{searchText}}”</button>
    </div>
    <iframe id="jspdf"
            ref="jspdf"
            name="jspdf"
            :src="`static/pdf/web/viewer.html?file=${pdfUrl}`"
            style="width:100%;height:calc(100vh - 54px);"
            frameborder="0"></iframe>
  </div>

</template>

<script>
export default {
  data () {
    return {
      // pdfUrl: `${location.origin}/static/test.pdf`,
      pdfUrl: `${location.origin}/vue-pdfpreview-demo/static/test.pdf`,
      pageNumber: 3,
      searchText: '工时登记',
      jspdfWindow: null
    }
  },
  mounted () {
    // 通过iframe的contentWindow对象来访问PDF.js提供的API
    this.jspdfWindow = this.$refs.jspdf.contentWindow
  },
  methods: {
    goPage () {
      if (this.jspdfWindow && this.jspdfWindow.PDFViewerApplication) {
        this.jspdfWindow.PDFViewerApplication.pdfViewer.currentPageNumber =
          this.pageNumber
      }
    },
    goSearch () {
      if (this.jspdfWindow && this.jspdfWindow.PDFViewerApplication) {
        const pdfViewer = this.jspdfWindow.PDFViewerApplication.pdfViewer
        this.jspdfWindow.PDFViewerApplication.pdfViewer.currentPageNumber =
          this.pageNumber
        this.jspdfWindow.PDFViewerApplication.pdfDocument.getPage(this.pageNumber).then(() => {
          pdfViewer.findController.executeCommand('find', {
            query: this.searchText,
            phraseSearch: true,
            caseSensitive: false,
            highlightAll: true,
            findPrevious: false
          })
        })
      }
    },
    openSearch () {
      if (this.jspdfWindow && this.jspdfWindow.PDFViewerApplication) {
        const findBar = this.jspdfWindow.PDFViewerApplication.findBar
        findBar.open()
        findBar.findField.value = this.searchText
        findBar.dispatchEvent(new Event('input'))
      }
    },
    before () { }
  }
}
</script>

<style>
</style>
