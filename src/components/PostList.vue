<script lang="ts" setup>
import PostDetail from '@/components/PostDetail.vue'

interface Post {
  title: string
  content: string
}

const info: Array<Post> = [
  { title: 'Este es el post 1', content: 'Texto de ejemplo 1' },
  { title: 'Este es el post 2', content: 'Texto de ejemplo 2' },
  { title: 'Este es el post 3', content: 'Texto de ejemplo 3' },
  { title: 'Este es el post 4', content: 'Texto de ejemplo 4' },
  { title: 'Este es el post 5', content: 'Texto de ejemplo 5' },
]
</script>

<template>
  <div class="container">
    <PostDetail v-for="elm in info" :key="elm.title" :content="elm.content" :title="elm.title" />
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  justify-content: space-around;
}
</style>
