<template>
  <div class='AnnotatedImage'>
    <div class='AnnotatedImage__Wrapper'>
      <img ref='img' class='AnnotatedImage__Image' :src="`./static/${image.image_path}`" />
      <svg xmlns="http://www.w3.org/2000/svg" version="1.1" width='640' height='480' @dblclick="$emit('rectCreated', $event, image.image_path)">
        <rect v-for='rect in image.rects' v-interactable
          :x="Math.min(rect.x1, rect.x2) || rect.x1"
          :y="Math.min(rect.y1, rect.y2) || rect.y1"
          :width="(Math.max(rect.x1, rect.x2) - Math.min(rect.x1, rect.x2)) || 5"
          :height="(Math.max(rect.y1, rect.y2) - Math.min(rect.y1, rect.y2)) || 5"
          fill='yellow'
          @dragmove="$emit('rectMoved', image.image_path, rect, $event)"
          @resizemove="$emit('rectResized', image.image_path, rect, $event)"
          @dblclick.stop="$emit('rectDeleted', image.image_path, rect)"
        />
      </svg>
   </div>
    <div class='AnnotatedImage__Rects'>
      <b>Name: {{image.image_path}}</b>
      <b>Rectangles: {{image.rects.length}}</b>
      <button @click='onSuggestClicked'>Suggest</button>
      <p v-for='rect in image.rects'>
        {{rect}} <button @click="$emit('rectDeleted', image.image_path, rect)">Delete</button>
      </p>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import interact from 'interactjs'

Vue.directive('interactable', {
  inserted: function (el) {
    interact(el)
      .draggable({})
      .resizable({ edges: { left: true, right: true, bottom: true, top: true }, margin: 4 })
      .on('dragmove', (event) => {
        event.target.dispatchEvent(new CustomEvent('dragmove', { detail: event }))
      })
      .on('resizemove', (event) => {
        event.target.dispatchEvent(new CustomEvent('resizemove', { detail: event }))
      })
  }
})

export default {
  props: ['image'],
  methods: {
    onSuggestClicked () {
      this.$emit('suggestRequested', this.$refs.img.src, this.image.image_path)
    }
  }
}
</script>

<style scoped>
.AnnotatedImage__Wrapper {
  display: flex;
  background-color: #E2E4E6;
  padding: 5px;
  border-radius: 5px;
  margin: 5px;
  position: relative;
}

.AnnotatedImage__Rects {
  text-align: center;
  font-size: 0.7em;
  background-color: #FFFFFF;
  padding: 5px;
  border-radius: 5px;
  margin: 5px;
}

.AnnotatedImage {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
}

.AnnotatedImage__Image {
  filter: grayscale(1);
}

svg {
  position: absolute;
}

rect {
  opacity: 0.4;
}
</style>
