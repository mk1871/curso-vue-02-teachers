<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'PostDetail',
  props: {
    title: {
      type: String,
      required: true,
    },
    content: {
      type: String,
      required: false,
      default: 'Este post no tiene contenido',
    },
  },
  setup(props) {
    return { props }
  },
})
</script>

<template>
  <div class="post">
    <h3>{{ props.title }}</h3>
    <p>{{ props.content }}</p>
  </div>
</template>

<style scoped>
.post {
  width: 300px;
  height: 200px;
  background-color: antiquewhite;
  border-radius: 10px;
  margin: 10px;
  padding: 10px;
}
</style>
