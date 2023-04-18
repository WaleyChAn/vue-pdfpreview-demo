<template>
  <iframe id="fd-jspdf"
          name="fd-jspdf"
          :src="`static/pdf/web/viewer.html?file=${pdfUrl}`"
          style="width:100%;height:100vh;"
          frameborder="0"></iframe>
</template>

<script>
export default {
  data () {
    return {
      pdfUrl: 'https://waleychan.github.io/vue-pdfpreview-demo/static/test.pdf'
    }
  }

}
</script>

<style>
</style>
