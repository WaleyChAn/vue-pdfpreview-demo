<template>
  <div>
    <div style='margin-bottom:10px'>
      <button @click='goPage()'>跳转到第{{pageNumber}}页</button>
      <button @click='goSearch()'
              style='margin-left:5px'>从第{{pageNumber}}页开始搜索“{{searchText}}”</button>
      <button @click='openSearch()'
              style='margin-left:5px'>弹出搜索框搜索“{{searchText}}”</button>
      <button @click='drawRect()'
              style='margin-left:5px'>功能位置码</button>
      <button @click='clear()'
              style='margin-left:5px'>清除位置码</button>
      <button @click='exportPage()'
              style='margin-left:5px'>导出该页图片</button>
    </div>
    <iframe id='jspdf'
            ref='jspdf'
            name='jspdf'
            :src='`static/pdf/web/viewer.html?file=${pdfUrl}`'
            style='width:100%; height:calc(100vh - 54px);'
            frameborder='0'></iframe>
  </div>

</template>

<script>
export default {
  data () {
    return {
      // pdfUrl: `${location.origin}/static/AA417RIS504CS3145DD-1@1-D-CAE-EN.pdf`,
      pdfUrl: `${location.origin}/vue-pdfpreview-demo/static/AA417RIS504CS3145DD-1@1-D-CAE-EN.pdf`,
      pageNumber: 1,
      searchText: '工时登记',
      isShowDrawRect: false,
      timeout: null,
      coordinate: [
        [
          [557, 175],
          [569, 175],
          [569, 206],
          [557, 206]
        ],
        [
          [904, 203],
          [934, 203],
          [934, 214],
          [904, 214]
        ],
        [
          [1019, 204],
          [1049, 204],
          [1049, 213],
          [1019, 213]
        ],
        [
          [1235, 203],
          [1265, 203],
          [1265, 214],
          [1235, 214]
        ],
        [
          [942, 234],
          [973, 234],
          [973, 247],
          [942, 247]
        ],
        [
          [1199, 236],
          [1231, 236],
          [1231, 248],
          [1199, 248]
        ],
        [
          [971, 283],
          [1000, 283],
          [1000, 292],
          [971, 292]
        ],
        [
          [1177, 282],
          [1208, 282],
          [1208, 293],
          [1177, 293]
        ],
        [
          [1066, 308],
          [1115, 308],
          [1115, 325],
          [1066, 325]
        ]
      ]
    }
  },
  mounted () {
    this.init()
  },
  methods: {
    init () {
      // 通过iframe的contentWindow对象来访问PDF.js提供的API
      let iframe = this.$refs.jspdf
      iframe.onload = () => {
        var checkInterval = setInterval(() => {
          this.PDFViewerApplication = iframe.contentWindow.PDFViewerApplication
          if (this.PDFViewerApplication) {
            clearInterval(checkInterval)
            // 获取iframe中的文档对象
            this.iframeDoc =
              iframe.contentDocument || iframe.contentWindow.document
            let toolbarViewerRight =
              this.iframeDoc.getElementById('toolbarViewerRight')
            toolbarViewerRight.style.display = 'none'
            // pdfViewer对象
            this.pdfViewer = this.PDFViewerApplication.pdfViewer
            // 获取findBar
            this.findBar = this.PDFViewerApplication.findBar
            console.log(this.PDFViewerApplication.eventBus)
            this.PDFViewerApplication.eventBus.on('scalechanging', (evt) => {
              if (this.isShowDrawRect) {
                this.timeout && clearTimeout(this.timeout)
                this.timeout = setTimeout(() => {
                  this.drawRect()
                }, 100)
              }
            })
          }
        }, 500)
      }
    },
    goPage () {
      this.pdfViewer.currentPageNumber = this.pageNumber
    },
    goSearch () {
      this.pdfViewer.currentPageNumber = this.pageNumber
      let pdfDoc = this.PDFViewerApplication.pdfDocument
      pdfDoc.getPage(this.pageNumber).then(() => {
        this.pdfViewer.findController.executeCommand('find', {
          query: this.searchText,
          phraseSearch: true,
          caseSensitive: false,
          highlightAll: true,
          findPrevious: false
        })
      })
    },
    openSearch () {
      this.findBar.open()
      this.findBar.findField.value = this.searchText
      this.findBar.dispatchEvent(new Event('input'))
    },
    drawRect () {
      this.renderPage('drawRect')
      this.isShowDrawRect = true
    },
    async renderPage (type) {
      // 获取pdf对象
      let pdfDoc = this.PDFViewerApplication.pdfDocument
      // 获取PDF.js Viewer的<canvas>元素
      const canvas = this.iframeDoc
        .getElementsByClassName('pdfViewer')[0]
        .getElementsByTagName('canvas')[this.pageNumber - 1]
      const context = canvas.getContext('2d')
      const page = await pdfDoc.getPage(this.pageNumber)
      // 获取页面视图以及canvas对象
      const scale = 1 // 缩放比例
      const viewport = page.getViewport({ scale })
      // 根据DPR调整Canvas元素大小和坐标值使图形清晰
      var dpr = 1.8
      canvas.width = viewport.width * dpr
      canvas.height = viewport.height * dpr
      context.scale(dpr, dpr)
      // 将PDF页面渲染到canvas上
      await page.render({
        canvasContext: context,
        viewport
      }).promise
      if (type === 'drawRect') {
        this.drawRectangle(context)
      }
    },
    drawRectangle (context) {
      // 绘制红色矩形并跟随缩放
      context.strokeStyle = 'red' // 红色边框
      context.lineWidth = 1 // 边框宽度
      this.coordinate.forEach((Rect) => {
        context.strokeRect(
          Rect[0][0],
          Rect[0][1],
          Rect[1][0] - Rect[0][0],
          Rect[2][1] - Rect[0][1]
        )
      })
    },
    clear () {
      this.isShowDrawRect = false
      this.renderPage()
    },
    exportPage () {
      const canvas = this.iframeDoc
        .getElementsByClassName('pdfViewer')[0]
        .getElementsByTagName('canvas')[this.pageNumber - 1]
      // 检测IE浏览器
      var isIE = /Trident|MSIE/.test(window.navigator.userAgent)
      var dataURL = null
      if (isIE) {
        // IE浏览器
        dataURL = canvas.toDataURL('image/jpeg', 1.0)
        var byteString = atob(dataURL.split(',')[1])
        var mimeString = dataURL.split(',')[0].split(':')[1].split('')[0]
        var ab = new ArrayBuffer(byteString.length)
        var ia = new Uint8Array(ab)
        for (var i = 0; i < byteString.length; i++) {
          ia[i] = byteString.charCodeAt(i)
        }
        var blob = new Blob([ab], { type: mimeString })
        window.navigator.msSaveBlob(blob, 'myChart.png')
      } else {
        // 其他浏览器
        // 调用toDataURL方法获取图数据URL
        dataURL = canvas.toDataURL('image/jpeg', 1.0)
        // 创建一个a标签并设置下载链接
        var downloadLink = document.createElement('a')
        downloadLink.href = dataURL
        downloadLink.download = 'myChart.png'

        // 触发点击事件下载图表
        downloadLink.click()
      }
    }
  }
}
</script>

<style>
#secondaryToolbarToggle {
  display: none;
}
</style>
