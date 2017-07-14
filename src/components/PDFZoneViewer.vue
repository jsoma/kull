<template>
<div class='pdf-zone-viewer' v-if="selections.length > 0">
  <h2>Your extraction code</h2>
  <p><strong>Edit the variable names and numbers below</strong> and you'll see it update to the right!</p>
  <textarea 
    @keyup="changed"
    ref="textarea"
    v-model="uzn"
    @blur="stopEditing"
    @focus="startEditing"></textarea>
  <p>This code uses <a href="https://github.com/jcushman/pdfquery" target="_blank">pdfquery</a>. You can just cut and paste if you want, but if you read the documentation it'll save you a lot of time.</p>
</div>
</template>

<script>
export default {
  props: ['selections', 'batchUpdateSelections', 'originalFilename', 'dimensions'],
  data () {
    return {
      uzn: null,
      editing: false
    }
  },
  computed: {
    downloadable () {
      const blob = new Blob([this.uzn], {type: 'octet/stream'})
      const url = window.URL.createObjectURL(blob)
      return url
    },
    filename () {
      return this.originalFilename.replace(/(.*)\..*/, '$1.uzn')
    }
  },
  mounted () {
    this.redrawUzn()
  },
  watch: {
    selections () {
      this.redrawUzn()
    }
  },
  methods: {
    startEditing () {
      this.editing = true
    },
    stopEditing () {
      this.editing = false
      this.redrawUzn()
    },
    redrawUzn () {
      if (this.editing) {
        return
      }
      if (this.selections.length === 0) {
        this.uzn = ''
        return
      }
      let lines = this.selections.map((s) => {
        let y0 = parseInt(this.dimensions.height - s.coordinates.top)
        let y1 = parseInt(this.dimensions.height - s.coordinates.top - s.coordinates.height)
        let bbox = [
          s.coordinates.left,
          Math.max(y0, y1),
          s.coordinates.left + s.coordinates.width,
          Math.min(y0, y1)
        ].join(',')
        let query = `LTPage[pageid=\\'${s.coordinates.page}\\'] LTTextLineHorizontal:overlaps_bbox("${bbox}")`
        return `\t('${s.name}', '${query}')`
      })

      let pretty = lines.join(',\n')
      this.uzn = `pdf = pdfquery.PDFQuery('${this.originalFilename}')\n\npdf.extract([\n\t('with_formatter', 'text'),\n${pretty}\n])`
    },
    changed (event) {
      console.log('CHANGED')
      let index = 0
      var selections = this.uzn.split('\n').map((line) => {
        let elements = line.match(/\('(.*)',.*pageid=\\'(\d+)\\'.*bbox\("(\d+),(\d+),(\d+),(\d+)"\)/)
        try {
          // 1 - name
          // 2 - page
          // 3 - left
          // 4 - top
          // 5 - right
          // 6 - bottom (?)
          var selection = this.selections[index]
          selection.coordinates.left = parseInt(elements[3])
          selection.coordinates.top = this.dimensions.height - parseInt(elements[4])
          selection.coordinates.width = parseInt(elements[5]) - parseInt(elements[3])
          selection.coordinates.height = Math.abs(parseInt(elements[6]) - parseInt(elements[4]))
          selection.name = elements[1]
          selection.page = elements[2]
          index++
          return selection
        } catch (err) {
          console.log(err)
          return null
        }
      }).filter((selection) => selection != null)

      this.batchUpdateSelections(selections)
    }
  }
}
</script>

<style scoped>
textarea {
  white-space: pre;
  overflow-wrap: normal;
  overflow-x: scroll;
  width: 100%;
  height: 300px;
  font-family: 'Courier New';
}
</style>
