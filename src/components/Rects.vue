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
      ctx.stroke()

      this.rects.forEach(({ x1, y1, x2, y2 }) => {
        if (!x2) {
          ctx.beginPath()
          ctx.globalAlpha = 1
          ctx.strokeStyle = 'yellow'
          ctx.lineWidth = 2

          ctx.moveTo(x1 - 80, y1)
          ctx.lineTo(x1 + 80, y1)

          ctx.moveTo(x1, y1 - 80)
          ctx.lineTo(x1, y1 + 80)
          ctx.stroke()

          ctx.beginPath()
          ctx.lineWidth = 1
          ctx.globalAlpha = 0.5
          for (let i = 0; i < 9; i++) {
            ctx.moveTo(x1 - 80 + i * 20, y1 - 80)
            ctx.lineTo(x1 - 80 + i * 20, y1 + 80)

            ctx.moveTo(x1 - 80, y1 - 80 + i * 20)
            ctx.lineTo(x1 + 80, y1 - 80 + i * 20)
          }
          ctx.stroke()
        } else {
          ctx.beginPath()
          ctx.globalAlpha = 0.5
          ctx.fillStyle = 'yellow'
          ctx.fillRect(x1, y1, x2 - x1, y2 - y1)
          ctx.stroke()
        }
      })
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
