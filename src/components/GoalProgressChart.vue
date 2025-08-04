<template>
  <div v-if="currentMiles !== null && goalMiles !== null && goalMiles > 0" class="goal-progress-chart">
    <canvas ref="chartCanvas" width="80" height="80"></canvas>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onUnmounted, nextTick } from 'vue';
import { Chart, ArcElement, Tooltip, Legend, DoughnutController } from 'chart.js';

// Register Chart.js components
Chart.register(ArcElement, Tooltip, Legend, DoughnutController);

const props = defineProps<{
  currentMiles: number;
  goalMiles: number;
}>();

const chartCanvas = ref<HTMLCanvasElement>();
let chart: Chart | null = null;

const createChart = () => {
  if (!chartCanvas.value) {
    return;
  }

  const ctx = chartCanvas.value.getContext('2d');
  if (!ctx) {
    return;
  }

  // Don't create chart if we don't have valid data
  if (props.currentMiles === null || props.goalMiles === null || props.goalMiles <= 0) {
    return;
  }

  let progress = Math.min((props.currentMiles / props.goalMiles) * 100, 100);
  let remaining = Math.max(100 - progress, 0);

  // Ensure we have some data to render
  if (progress === 0 && remaining === 0) {
    progress = 0.1;
    remaining = 99.9;
  }

  // Destroy existing chart if it exists
  if (chart) {
    chart.destroy();
  }

  chart = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels: ['Progress', 'Remaining'],
      datasets: [{
        data: [progress, remaining],
        backgroundColor: [
          '#4CAF50', // Green for progress
          '#e0e0e0'  // Darker gray for remaining - more visible
        ],
        borderWidth: 0
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      cutout: '70%',
      plugins: {
        legend: {
          display: false
        },
        tooltip: {
          enabled: false
        }
      },
      elements: {
        arc: {
          borderRadius: 4
        }
      }
    }
  });

};

const updateChart = () => {
  // Always recreate the chart for reliable updates
  if (chart) {
    chart.destroy();
    chart = null;
  }
  createChart();
};

onUnmounted(() => {
  if (chart) {
    chart.destroy();
    chart = null;
  }
});

watch(() => [props.currentMiles, props.goalMiles], () => {
  nextTick(() => {
    // Destroy chart if data becomes invalid
    if (props.currentMiles === null || props.goalMiles === null || props.goalMiles <= 0) {
      if (chart) {
        chart.destroy();
        chart = null;
      }
      return;
    }
    
    // Always recreate the chart for reliable updates
    updateChart();
  });
}, { deep: true, immediate: true });
</script>

<style scoped>
.goal-progress-chart {
  width: 80px;
  height: 80px;
  margin: 0 auto;
}
</style> 