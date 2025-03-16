<template>
  <div class="step">
    <h2>选择设备类型</h2>
    <div class="device-grid">
      <button
        v-for="(device, index) in devices"
        :key="index"
        @click="$emit('device-selected', index)"
        :class="{ 'selected': selectedDevice === index }"
        class="device-btn"
      >
        {{ index + 1 }}. {{ device }}
      </button>
    </div>
    <div v-if="error" class="error">{{ error }}</div>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from 'vue';

const props = defineProps({
  devices: {
    type: Array,
    required: true
  },
  selectedDevice: {
    type: Number,
    default: null
  },
  error: {
    type: String,
    default: null
  }
});

const emit = defineEmits(['device-selected']);
</script>

<style scoped>
.step {
  background: white;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
}

.device-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.5rem;
  margin: 1rem 0;
}

.device-btn {
  padding: 0.8rem;
  border: 2px solid #cec9c9;
  border-radius: 6px;
  background: rgb(205, 214, 212);
  cursor: pointer;
  text-align: left;
  transition: all 0.2s;
  font-size: 0.9rem;
}

.device-btn:hover {
  border-color: #42b983;
  transform: translateY(-2px);
}

.device-btn.selected {
  border-color: #42b983;
  background: #f0f9f4;
}

.error {
  color: #dc3545;
  margin-top: 0.5rem;
  padding: 0.3rem;
  background: #f8d7da;
  border-radius: 4px;
  font-size: 0.9rem;
}
</style>