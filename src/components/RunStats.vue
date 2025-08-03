<template>
  <div v-if="milesRun !== null" class="run-stats">
    <article>
      <header>
        <h2 class="stats-header">Your Running Statistics</h2>
      </header>
      <div class="stats-grid">
        <!-- First row: Total Miles and Yearly Goal Progress -->
        <div class="stat-card">
          <h3>Total Miles</h3>
          <p class="stat-value">{{ milesRun?.toFixed(1) || '0.0' }}</p>
          <p class="stat-label">miles run this year</p>
        </div>
        
        <div v-if="goal" class="stat-card">
          <h3>Goal Progress</h3>
          <div class="goal-progress-container">
            <div class="goal-progress-text">
              <p class="stat-value">{{ goalProgress }}%</p>
              <p class="stat-label">{{ milesRun?.toFixed(1) || '0.0' }} / {{ goal }} miles</p>
            </div>
            <GoalProgressChart :current-miles="milesRun" :goal-miles="goal" />
          </div>
        </div>

        <!-- Second row: Average Miles Per Day and Projected Year-End -->
        <div class="stat-card">
          <h3>Daily Average</h3>
          <p class="stat-value">{{ averageMilesPerDay.toFixed(2) }}</p>
          <p class="stat-label">miles per day</p>
        </div>

        <div class="stat-card">
          <h3>Projected Total</h3>
          <p class="stat-value">{{ projectedYearEnd.toFixed(1) }}</p>
          <p class="stat-label">miles by year end</p>
        </div>

        <!-- Third row: Rest Days or Warning -->
        <div v-if="goal && !isBehindSchedule && projectedYearEnd >= goal" class="stat-card rest-days-card">
          <h3>Rest Days Available</h3>
          <p class="stat-value">{{ restDaysText }}</p>
          <p class="stat-label">you could take off</p>
        </div>
        
        <!-- Warning card when behind schedule -->
        <div v-if="goal && isBehindSchedule" class="stat-card warning-card">
          <h3>Goal Status</h3>
          <p class="stat-value">{{ projectedYearEnd.toFixed(1) }}</p>
          <p class="stat-label">projected year end (goal: {{ goal }})</p>
        </div>
      </div>
    </article>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import GoalProgressChart from './GoalProgressChart.vue';

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

const isBehindSchedule = computed(() => {
  if (!props.goal || !props.milesRun) return false;
  
  const remainingDays = daysInYear - daysSoFar.value;
  const remainingMiles = props.goal - props.milesRun;
  const requiredDailyMiles = remainingMiles / remainingDays;
  const currentDailyMiles = props.milesRun / daysSoFar.value;
  
  // If we need more miles per day than we're currently averaging, we're behind
  return currentDailyMiles < requiredDailyMiles;
});
</script>

<style scoped>
.run-stats {
  max-width: 740px;
  margin: 0 auto;
}

.stats-header {
  font-size: 1.4rem;
  font-weight: bold;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 0.85rem;
  margin-top: 1rem;
}

.stat-card {
  background-color: #f5f5f5;
  padding: 1rem;
  border-radius: 8px;
  text-align: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.stat-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.stat-card h3 {
  margin-top: 0;
  margin-bottom: 0.425rem;
  color: #333;
  font-size: 0.935rem;
}

.stat-value {
  font-size: 2.125rem;
  font-weight: bold;
  color: #4CAF50;
  margin-bottom: 0.2125rem;
  line-height: 1;
}

.stat-label {
  font-size: 0.765rem;
  color: #666;
  margin: 0;
}

.goal-progress-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}

.goal-progress-text {
  flex: 1;
  text-align: left;
}

.goal-progress-text .stat-value {
  margin-bottom: 0.2125rem;
}

.goal-progress-text .stat-label {
  margin: 0;
}

.rest-days-card {
  background-color: #e8f5e9;
  color: #2e7d32;
}

.rest-days-card .stat-value {
  color: #2e7d32;
}

.warning-card {
  background-color: #fff3cd;
  color: #856404;
}

.warning-card .stat-value {
  color: #856404;
}

/* Mobile responsiveness */
@media (max-width: 768px) {
  .stats-grid {
    grid-template-columns: 1fr;
    gap: 0.64rem;
  }
  
  .stat-card {
    padding: 1.0625rem;
  }
  
  .stat-value {
    font-size: 1.7rem;
  }
  
  .stat-card h3 {
    font-size: 0.85rem;
  }
  
  .goal-progress-container {
    flex-direction: column;
    gap: 0.5rem;
  }
  
  .goal-progress-text {
    text-align: center;
  }
}

@media (min-width: 769px) and (max-width: 1024px) {
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>