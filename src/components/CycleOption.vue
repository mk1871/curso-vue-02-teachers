<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'CycleOption',
  created() {
    console.log('Created')
  },

  mounted() {
    console.log('Mounted')
  },

  updated() {
    console.log('Updated')
  },

  unmounted() {
    console.log('Unmounted')
  },
})
</script>

<template>
  <div>Hola, mundo</div>
</template>

<style scoped></style>
