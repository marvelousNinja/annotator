<template>
  <div id='app'>
    <button @click='downloadJson'>Export as JSON</button>
    <a ref='downloadLink' style='display: none;' download='data.json'></a>
    <AnnotatedImage
      v-for='image in visibleImages'
      :image='image'
      @suggestRequested='suggestRequested'
      @rectMoved='rectMoved'
      @rectResized='rectResized'
      @rectCreated='rectCreated'
      @rectDeleted='rectDeleted'
    />
  </div>
</template>

<script>
import AnnotatedImage from './components/AnnotatedImage'
import { concat, times, map, pipe, evolve, filter, min, max, prepend, findIndex, whereEq, lensPath, over, without, add, values, clamp } from 'ramda'
import axios from 'axios'
import { debounce } from 'lodash'

const padNum = num => ('00' + num).slice(-3)
const imageNames = map(num => `resized${num}.png`, times(padNum, 180))

export default {
  components: {
    AnnotatedImage
  },
  data () {
    return {
      images: JSON.parse(localStorage.getItem('images')) || map(name => ({
        image_path: name,
        rects: []
      }), imageNames),
      page: 0
    }
  },
  computed: {
    visibleImages () { return this.images.slice(0, 10) }
  },
  watch: {
    images: debounce((val) => localStorage.setItem('images', JSON.stringify(val)), 300)
  },
  methods: {
    downloadJson () {
      const jsonData = pipe(
        filter(img => img.rects.length > 0),
        map(evolve({
          rects: map(({ x1, x2, y1, y2 }) => ({
            x1: min(x1, x2),
            x2: max(x1, x2),
            y1: min(y1, y2),
            y2: max(y1, y2)
          }))
        }))
      )(this.images)

      this.$refs.downloadLink.href = 'data:text/json;charset=utf-8,' + encodeURIComponent(JSON.stringify(jsonData, null, 2))
      this.$refs.downloadLink.click()
    },
    rectCreated (event, imagePath) {
      const x = event.offsetX
      const y = event.offsetY >= 2 ? event.offsetY - 2 : event.offsetY
      const index = findIndex(whereEq({ image_path: imagePath }), this.images)

      this.$set(this, 'images', over(
        lensPath([index, 'rects']),
        prepend({
          x1: clamp(0, 640, x - 20),
          y1: clamp(0, 480, y - 20),
          x2: clamp(0, 640, x + 20),
          y2: clamp(0, 480, y + 20)
        }),
        this.images
      ))
    },
    rectDeleted (imagePath, rect) {
      const index = findIndex(whereEq({ image_path: imagePath }), this.images)
      this.$set(this, 'images', over(
        lensPath([index, 'rects']),
        without([rect]),
        this.images
      ))
    },
    suggestRequested (url, imagePath) {
      axios.post('http://localhost:80/segment', { url })
        .then(({ data }) => {
          const index = findIndex(whereEq({ image_path: imagePath }), this.images)
          this.$set(this, 'images', over(
            lensPath([index, 'rects']),
            concat(data),
            this.images
          ))
        })
    },
    rectMoved (imagePath, rect, event) {
      const index = findIndex(whereEq({ image_path: imagePath }), this.images)
      const rects = values(this.images[index].rects)
      const rectIndex = findIndex(whereEq(rect), rects)

      this.$set(this, 'images', over(
        lensPath([index, 'rects', rectIndex]),
        evolve({
          x1: pipe(add(event.detail.dx), clamp(0, 640)),
          x2: pipe(add(event.detail.dx), clamp(0, 640)),
          y1: pipe(add(event.detail.dy), clamp(0, 480)),
          y2: pipe(add(event.detail.dy), clamp(0, 480))
        }),
        this.images
      ))
    },
    rectResized (imagePath, rect, event) {
      const index = findIndex(whereEq({ image_path: imagePath }), this.images)
      const rects = values(this.images[index].rects)
      const rectIndex = findIndex(whereEq(rect), rects)

      this.$set(this, 'images', over(
        lensPath([index, 'rects', rectIndex]),
        evolve({
          x1: pipe(add(event.detail.deltaRect.left), clamp(0, 640)),
          x2: pipe(add(event.detail.deltaRect.right), clamp(0, 640)),
          y1: pipe(add(event.detail.deltaRect.top), clamp(0, 480)),
          y2: pipe(add(event.detail.deltaRect.bottom), clamp(0, 480))
        }),
        this.images
      ))
    }
  }
}
</script>

<style>
* {
  margin: 0px;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2C3E50;
  background-color: #838C91;
}
</style>
