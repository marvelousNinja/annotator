<template>
  <!-- TODO AS: Nasty hack, oh noes-->
  <canvas ref='canvas' width='640' height='480'></canvas>
</template>

<script>
export default {
  props: ['rects'],
  mounted () {
    this.redraw()
  },
  watch: {
    rects () {
      this.redraw()
    }
  },
  methods: {
    redraw () {
      const canvas = this.$refs.canvas
      const ctx = canvas.getContext('2d')

      ctx.beginPath()
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      ctx.fillStyle = 'purple'
      // ctx.strokeStyle = 'green'
      ctx.globalAlpha = 0.5
      this.rects.forEach(({ x1, y1, x2, y2 }) => {
        ctx.fillRect(x1, y1, (x2 - x1) || 5, (y2 - y1) || 5)
      })

      ctx.stroke()
    }
  }
}
</script>

<style>
canvas {
  pointer-events: none;
  position: absolute;
}
</style>
