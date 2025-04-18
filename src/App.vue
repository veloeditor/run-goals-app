<template>
  <main class="container">
    <h1>RunMetrics</h1>
    <RunInputForm v-model:miles="milesRun" v-model:goal="runningGoal" />
    <RunStats :miles-run="milesRun" :goal="runningGoal" />
  </main>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import RunInputForm from './components/RunInputForm.vue';
import RunStats from './components/RunStats.vue';

const milesRun = ref<number | null>(null);
const runningGoal = ref<number | null>(null);

onMounted(() => {
  const savedGoal = localStorage.getItem('runningGoal');
  if (savedGoal) {
    runningGoal.value = parseInt(savedGoal);
  }
});
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
