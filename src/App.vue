<template>
  <main class="container">
    <h1>RunMetrics</h1>
    <RunInputForm v-model:miles="milesRun" />
    <RunStats :miles-run="milesRun" />
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import RunInputForm from './components/RunInputForm.vue';
import RunStats from './components/RunStats.vue';

const milesRun = ref<number | null>(null);
</script>

<style>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

h1 {
  text-align: center;
  margin-bottom: 2rem;
}
</style>
