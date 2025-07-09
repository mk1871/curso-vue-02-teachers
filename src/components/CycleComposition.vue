<script lang="ts" setup>
import { onMounted, onUpdated } from 'vue'

onMounted(() => {
  console.log('Mounted')
})

onUpdated(() => {
  console.log('Updated')
})
</script>

<template>
  <div>Hola, Mundo</div>
</template>

<style scoped></style>
