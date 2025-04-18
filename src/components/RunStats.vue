<template>
  <div v-if="milesRun !== null" class="run-stats">
    <article>
      <header>
        <h2>Your Running Statistics</h2>
      </header>
      <div class="stats-content">
        <p><strong>Total Miles Run This Year:</strong> {{ milesRun.toFixed(1) }}</p>
        <p v-if="goal"><strong>Yearly Goal Progress:</strong> {{ goalProgress }}% ({{ milesRun.toFixed(1) }} / {{ goal
        }} miles)</p>
        <p><strong>Average Miles Per Day:</strong> {{ averageMilesPerDay.toFixed(2) }}</p>
        <p><strong>Projected Year-End Total:</strong> {{ projectedYearEnd.toFixed(1) }}</p>
        <p v-if="goal && projectedYearEnd >= goal" class="rest-days">
          <strong>You could take off:</strong> {{ restDaysText }}
        </p>
      </div>
    </article>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
  milesRun: number | null;
  goal: number | null;
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

const goalProgress = computed(() => {
  if (!props.goal || !props.milesRun) return 0;
  return ((props.milesRun / props.goal) * 100).toFixed(1);
});

const restDaysText = computed(() => {
  if (!props.goal || !props.milesRun || projectedYearEnd.value < props.goal) return '0 days';

  const remainingDays = daysInYear - daysSoFar.value;
  const remainingMiles = props.goal - props.milesRun;
  const requiredDailyMiles = remainingMiles / remainingDays;
  const currentDailyMiles = props.milesRun / daysSoFar.value;

  if (currentDailyMiles <= requiredDailyMiles) return '0 days';

  const possibleRestDays = Math.floor((currentDailyMiles - requiredDailyMiles) * remainingDays / requiredDailyMiles);

  if (possibleRestDays < 7) {
    return `${possibleRestDays} day${possibleRestDays !== 1 ? 's' : ''}`;
  } else {
    const weeks = (possibleRestDays / 7).toFixed(1);
    return `${weeks} week${weeks === '1.0' ? '' : 's'}`;
  }
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
  gap: 0.42rem;
}

article {
  padding: 1.5rem;
}

.rest-days {
  color: #4CAF50;
  font-weight: bold;
}
</style>