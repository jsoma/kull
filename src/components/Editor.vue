<template>
  <div class='editor'>
    <div class='sidebar'>
      <h1>Kull</h1>
      <p><strong>A tool to interactively select text regions of PDFs and images.</strong> Use with <a href="https://github.com/jcushman/pdfquery">PDFQuery</a> (Python/PDF) or to make UZN/OCR zone files for <a href="https://github.com/tesseract-ocr/tesseract">tesseract</a> (image-to-text).</p>
      <p>More details (maybe) at <a href="https://github.com/jsoma/kull">https://github.com/jsoma/kull</a>.</p>
      <Uploader :notify="newFile"></Uploader>
      <p v-if="name"><strong>Click and drag on the image to the right</strong> to form selection areas.</p>

      <ZoneViewer :selections="selections" class='zone-viewer' :batchUpdateSelections="batchUpdateSelections" :originalFilename="name" v-if="src"></ZoneViewer>
      <PDFZoneViewer :dimensions="pdfDimensions" :selections="selections" class='zone-viewer' :batchUpdateSelections="batchUpdateSelections" :originalFilename="name" v-if="arrayBuffer"></PDFZoneViewer>
    </div>
    <div class='content'>
      <Annotator :src="src" :setPdfSize="setPdfSize" :arrayBuffer="arrayBuffer" :name="name" :selections="selections" :addSelection="addSelection"></Annotator>
    </div>
  </div>
</template>

<script>
import Uploader from '@/components/Uploader'
import Annotator from '@/components/Annotator'
import ZoneViewer from '@/components/ZoneViewer'
import PDFZoneViewer from '@/components/PDFZoneViewer'
import randomColor from 'randomcolor'

export default {
  name: 'editor',
  components: {
    Uploader,
    Annotator,
    ZoneViewer,
    PDFZoneViewer
  },
  data () {
    return {
      src: null,
      arrayBuffer: null,
      name: '',
      selections: [],
      pdfDimensions: {
        height: 0,
        width: 0
      }
    }
  },
  methods: {
    batchUpdateSelections: function (selections) {
      this.selections = selections
    },
    setPdfSize: function (width, height) {
      this.pdfDimensions = {
        width: width,
        height: height
      }
    },
    addSelection: function (coords) {
      if (coords.height === 0 || coords.width === 0) {
        return
      }

      this.selections.push({
        id: +new Date(),
        coordinates: {
          top: coords.top,
          left: coords.left,
          height: coords.height,
          width: coords.width,
          page: coords.page,
          pageOffset: coords.pageOffset
        },
        color: randomColor({format: 'rgb'}),
        name: 'Unnamed' + this.selections.length
      })
    },
    newFile: function (data) {
      this.name = data.name
      this.src = data.src
      this.arrayBuffer = data.arrayBuffer
      this.selections = []
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

.editor {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;

  margin-top: 60px;
}

.sidebar {
  position: absolute;
  width: 400px;
  top: 0;
  left: 0;
  padding: 10px;
}

.content {
  position: absolute;
  width: auto;
  margin-left: 30px;
  top: 0px;
  left: 410px;
}

textarea {
  -webkit-box-sizing: border-box;
   -moz-box-sizing: border-box;
        box-sizing: border-box;
  font-size: 1em;
}

h2 {
  background: #FFF800;
  padding: 10px 25px;
}

h1 {
  background: #C50080;
  padding: 10px 25px;
  color: white;
}
</style>
