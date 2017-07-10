<template>
  <div class='selection-box' :style="styleObject">
    <span>{{ name }}</span>
  </div>
</template>

<script>
export default {
  name: 'area-select',
  props: ['coordinates', 'color', 'name', 'active'],
  computed: {
    styleObject: function () {
      if (this.height === 0 || this.width === 0) {
        return {
          display: 'none'
        }
      }
      return {
        left: this.coordinates.left + this.coordinates.pageOffset.left + 'px',
        top: this.coordinates.top + this.coordinates.pageOffset.top + 'px',
        width: this.coordinates.width + 'px',
        height: this.coordinates.height + 'px',
        border: 'solid ' + this.color + ' 1px',
        background: this.color.replace(/\)$/, ', 0.05)').replace('rgb(', 'rgba(')
      }
    }
  }
}
</script>

<style scoped>
.selection-box {
  position: absolute;
  pointer-events: none;
  text-align: center;
  z-index: 1000;
}

span {
  color: black;
  display: inline;
  position: relative;
  font-weight: bold;
  font-size: 12px;
  text-shadow: white 0px 0px 2px, white 0px 0px 2px, white 0px 0px 5px, white 0px 0px 5px, white 0px 0px 30px, white 0px 0px 30px, white 0px 0px 30px, white 0px 0px 30px, white 0px 0px 30px, white 0px 0px 60px, white 0px 0px 60px, white 0px 0px 60px;
}
</style>
