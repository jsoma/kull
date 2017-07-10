<template>
<div class='zone-viewer' v-if="selections.length > 0">
  <h2>Your zone file</h2>
  <p><strong>Edit the code below</strong> and it'll update on the right!</p>
  <textarea 
    @keyup="changed"
    ref="textarea"
    v-model="uzn"
    @blur="stopEditing"
    @focus="startEditing"
  ></textarea>
  <a :href="downloadable" :download="filename">Download .uzn file</a>
</div>
</template>

<script>
export default {
  name: 'zone-viewer',
  props: ['selections', 'batchUpdateSelections', 'originalFilename'],
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
      let lines = this.selections.map((s) => {
        return `${s.coordinates.left} ${s.coordinates.top} ${s.coordinates.width} ${s.coordinates.height} ${s.name}`
      })

      this.uzn = lines.join('\n')
    },
    changed (event) {
      var selections = this.uzn.split('\n').map((line, index) => {
        try {
          let elements = line.match(/(\d+) (\d+) (\d+) (\d+) (.*)/)
          var selection = this.selections[index]
          selection.coordinates.left = parseInt(elements[1])
          selection.coordinates.top = parseInt(elements[2])
          selection.coordinates.width = parseInt(elements[3])
          selection.coordinates.height = parseInt(elements[4])
          selection.name = elements[5]
          return selection
        } catch (err) {
          return null
        }
      }).filter((selection) => selection != null)
      console.log('updating with', selections)
      this.batchUpdateSelections(selections)
    }
  }
}
</script>

<style>
textarea {
  white-space: pre;
  overflow-wrap: normal;
  overflow-x: scroll;
  width: calc(100% - 20px);
  min-height: 200px;
}
</style>
