<template>
  <div id="app">
    <pdf-review :url="url"></pdf-review>
  </div>
</template>

<script>
import pdfReview from './components/PdfReview.vue'

export default {
  name: 'app',
  components: {
    pdfReview
  },
  data () {
    return {
      url: 'http://118.24.174.95/api/mvp-manager/file/download/900A03F555FF487296EB5C9E918924BE'
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
