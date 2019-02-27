<template>
  <div id="app">
    <pdf-preview :url="url"></pdf-preview>
  </div>
</template>

<script>
import pdfPreview from './components/PdfPreview.vue'

export default {
  name: 'app',
  components: {
    pdfPreview
  },
  data () {
    return {
      url: 'http://118.24.174.95/api/mvp-manager/file/download/5F87A68CA98D4B0092E9C6EA4B528DB6'
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
