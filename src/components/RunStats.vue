<template>
  <div v-if="milesRun !== null" class="run-stats">
    <article>
      <header>
        <h2>Your Running Statistics</h2>
      </header>
      <div class="stats-content">
        <p><strong>Total Miles Run This Year:</strong> {{ milesRun.toFixed(1) }}</p>
        <p><strong>Average Miles Per Day:</strong> {{ averageMilesPerDay.toFixed(2) }}</p>
        <p><strong>Projected Year-End Total:</strong> {{ projectedYearEnd.toFixed(1) }}</p>
      </div>
    </article>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
  milesRun: number | null;
}>();

const daysInYear = 365;
const daysSoFar = computed(() => {
  const now = new Date();
  const start = new Date(now.getFullYear(), 0, 0);
  const diff = now.getTime() - start.getTime();
  return Math.floor(diff / (1000 * 60 * 60 * 24));
});

const averageMilesPerDay = computed(() => {
  if (props.milesRun === null) return 0;
  return props.milesRun / daysSoFar.value;
});

const projectedYearEnd = computed(() => {
  if (props.milesRun === null) return 0;
  return averageMilesPerDay.value * daysInYear;
});
</script>

<style scoped>
.run-stats {
  max-width: 600px;
  margin: 0 auto;
}

.stats-content {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

article {
  padding: 1.5rem;
}
</style>