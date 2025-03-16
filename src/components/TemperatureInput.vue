<template>
  <div class="step">
    <h2>输入温度数据 (℃)</h2>
    <div class="input-group">
      <div class="input-item">
        <label>T0（环境温度）:</label>
        <input type="number" :value="T0" @input="updateT0($event.target.value)" step="0.1">
      </div>
      <div class="input-item">
        <label>T1（发热点）:</label>
        <input type="number" :value="T1" @input="updateT1($event.target.value)" step="0.1">
      </div>
      <div class="input-item">
        <label>T2（正常点）:</label>
        <input type="number" :value="T2" @input="updateT2($event.target.value)" step="0.1">
      </div>
    </div>
    <div class="button-group">
      <button @click="$emit('go-back')" class="btn secondary">返回</button>
      <button @click="$emit('calculate')" class="btn primary">计算</button>
    </div>
    <div v-if="error" class="error">{{ error }}</div>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from 'vue';

const props = defineProps({
  T0: {
    type: Number,
    default: null
  },
  T1: {
    type: Number,
    default: null
  },
  T2: {
    type: Number,
    default: null
  },
  error: {
    type: String,
    default: null
  }
});

const emit = defineEmits(['go-back', 'calculate', 'update:T0', 'update:T1', 'update:T2']);

const updateT0 = (value) => {
  emit('update:T0', parseFloat(value));
};

const updateT1 = (value) => {
  emit('update:T1', parseFloat(value));
};

const updateT2 = (value) => {
  emit('update:T2', parseFloat(value));
};
</script>

<style scoped>
.step {
  background: white;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
}

.input-group {
  display: grid;
  gap: 0.8rem;
  margin: 1.5rem 0;
}

.input-group label {
            flex: 0 0 40%;
            padding-right: 1rem;
            text-align: right;
}
.input-item {
  display: flex;
  align-items: center;
  gap: 0.8rem;
}

input {
  padding: 0.4rem;
  border: 2px solid #e0e0e0;
  border-radius: 4px;
  width: 70%;
  font-size: 0.9rem;
}

.button-group {
  display: flex;
  gap: 0.8rem;
  justify-content: center;
  margin-top: 1.5rem;
}

.btn {
  padding: 0.6rem 1.2rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 0.9rem;
}

.primary {
  background: #42b983;
  color: white;
}

.secondary {
  background: #e0e0e0;
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