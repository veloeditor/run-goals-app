<template>
  <div class="run-input-form">
    <div class="goal-section">
      <template v-if="!storedGoal">
        <label for="goal-miles">Set your yearly running goal (miles):</label>
        <div class="goal-input-container">
          <input type="number" id="goal-miles" v-model="goalMiles" placeholder="Enter goal miles">
          <button @click="saveGoal" :disabled="!goalMiles">Save Goal</button>
        </div>
      </template>
      <template v-else>
        <div class="goal-display">
          <span>Yearly Goal: {{ storedGoal }} miles</span>
          <button @click="editGoal">Edit Goal</button>
        </div>
      </template>
    </div>

    <label for="miles-run">Enter your total miles run this year:</label>
    <input type="number" id="miles-run" v-model="milesRun" placeholder="Enter miles" @input="updateMiles">
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';

const props = defineProps<{
  miles: number | null;
  goal: number | null;
}>();

const emit = defineEmits<{
  (e: 'update:miles', value: number | null): void;
  (e: 'update:goal', value: number | null): void;
}>();

const milesRun = ref<number | null>(null);
const goalMiles = ref<number | null>(null);
const storedGoal = ref<number | null>(null);

const updateMiles = () => {
  emit('update:miles', milesRun.value);
};

const saveGoal = () => {
  if (goalMiles.value) {
    localStorage.setItem('runningGoal', goalMiles.value.toString());
    storedGoal.value = goalMiles.value;
    emit('update:goal', goalMiles.value);
    goalMiles.value = null;
  }
};

const editGoal = () => {
  goalMiles.value = storedGoal.value;
  storedGoal.value = null;
  emit('update:goal', null);
};

onMounted(() => {
  const savedGoal = localStorage.getItem('runningGoal');
  if (savedGoal) {
    storedGoal.value = parseInt(savedGoal);
    emit('update:goal', storedGoal.value);
  }
});

// Sync with parent component
watch(() => props.miles, (newValue) => {
  milesRun.value = newValue;
});

watch(() => props.goal, (newValue) => {
  if (newValue !== storedGoal.value) {
    storedGoal.value = newValue;
  }
});
</script>

<style scoped>
.run-input-form {
  margin-bottom: 1.2rem;
}

.goal-section {
  margin-bottom: 2rem;
}

.goal-input-container {
  display: flex;
  gap: 1rem;
  align-items: center;
  margin-top: 0.5rem;
}

.goal-display {
  display: flex;
  gap: 1rem;
  align-items: center;
}

input {
  margin-top: 0.5rem;
}

#miles-run {
  width: 200px;
}

button {
  padding: 0.25rem 0.75rem;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9rem;
  position: relative;
  bottom: 4px;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

button:hover:not(:disabled) {
  background-color: #45a049;
}
</style>