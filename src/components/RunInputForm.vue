<template>
  <div class="run-input-form">
    <label for="miles-run">Enter your total miles run this year:</label>
    <input type="number" id="miles-run" v-model="milesRun" placeholder="Enter miles" @input="updateMiles">
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const milesRun = ref<number | null>(null);

const emit = defineEmits<{
  (e: 'update:miles', value: number | null): void;
}>();

const updateMiles = () => {
  emit('update:miles', milesRun.value);
};
</script>

<style scoped>
.run-input-form {
  margin-bottom: 2rem;
}

input {
  margin-top: 0.5rem;
}
</style>