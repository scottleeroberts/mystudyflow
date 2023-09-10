<template>
  <div class="container text-center py-5">
    <div v-if="!isRunning" class="mb-3">
      <input
        type="text"
        v-model="pomodoroName"
        placeholder="Enter Pomodoro Name"
        class="form-control text-center"
      />
    </div>
    <div v-else class="mb-3 h3">{{ pomodoroName }}</div>
    <div class="h2 mb-3">{{ formattedTime }}</div>
    <button
      v-if="!isRunning"
      @click="startPomodoro"
      class="btn btn-success me-2"
    >
      Start
    </button>
    <button
      v-else
      @click="stopPomodoro"
      class="btn btn-danger me-2"
    >
      Stop
    </button>
    <button
      @click="clearHistory"
      class="btn btn-warning"
    >
      Clear History
    </button>
    <div v-if="successMessage" class="alert alert-success mt-4">
      {{ successMessage }}
    </div>
    <div class="mt-5">
      <h2>Pomodoro History</h2>
      <ul class="list-group">
        <li v-for="(item, index) in pomodoroHistory" :key="index" class="list-group-item">
          <span class="fw-bold">{{ item.name }}</span> |
          <span>Start: {{ item.start }}</span> |
          <span>End: {{ item.end }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const pomodoroName = ref('');
const duration = ref(25 * 60); // 25 minutes in seconds
const isRunning = ref(false);
const pomodoroHistory = ref(JSON.parse(localStorage.getItem('pomodoroHistory') || '[]'));
const successMessage = ref('');
let timer = null;

const formattedTime = computed(() => {
  const minutes = Math.floor(duration.value / 60);
  const seconds = duration.value % 60;
  return `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
});

function startPomodoro() {
  if (!pomodoroName.value.trim()) {
    successMessage.value = 'Please enter a name for the Pomodoro.';
    return;
  }
  successMessage.value = ''; // Clear previous success message
  isRunning.value = true;
  timer = setInterval(() => {
    if (duration.value > 0) {
      duration.value--;
    } else {
      completePomodoro();
    }
  }, 1000);
}

function stopPomodoro() {
  isRunning.value = false;
  clearInterval(timer);
  duration.value = 25 * 60; // Reset to 25 minutes
}

function completePomodoro() {
  const endTime = new Date();
  pomodoroHistory.value.push({
    name: pomodoroName.value,
    start: new Date().toLocaleString(),
    end: endTime.toLocaleString(),
  });
  localStorage.setItem('pomodoroHistory', JSON.stringify(pomodoroHistory.value));
  isRunning.value = false;
  duration.value = 25 * 60; // Reset to 25 minutes
  pomodoroName.value = ''; // Clear name for next Pomodoro
  successMessage.value = 'Pomodoro Complete!';
}

function clearHistory() {
  pomodoroHistory.value = [];
  localStorage.setItem('pomodoroHistory', JSON.stringify([]));
  successMessage.value = 'History cleared successfully.';
}
</script>

<style scoped>
</style>
