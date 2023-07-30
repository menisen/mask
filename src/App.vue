<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue'
import {ref} from 'vue'
let input = ref('')
</script>

<template>
  <h1>Mask</h1>
  <HelloWorld mask="+1 (###) ###-####" v-model:value="input" />
</template>

<style scoped>
</style>
