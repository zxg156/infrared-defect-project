<template>
  <div class="step result">
    <h2>分析结果</h2>
    <table class="result-table">
      <tbody>
        <tr>
          <th>所选设备</th>
          <td>{{ device }}</td>
        </tr>
        <tr>
          <th>δ值</th>
          <td>{{ delta }}%</td>
        </tr>
        <tr :class="defectClass">
          <th>缺陷等级</th>
          <td>{{ defectLevel }}</td>
        </tr>
        <tr>
          <th>日期和时间</th>
          <td>{{ timestamp }}</td>
        </tr>
      </tbody>
    </table>
    <button @click="$emit('reset')" class="btn primary">重新开始</button>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits, computed } from 'vue';

const props = defineProps({
  delta: {
    type: String,
    default: null
  },
  defectLevel: {
    type: String,
    default: ''
  },
  defectConditions: {
    type: String,
    default: ''
  },
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
  device: {
    type: String,
    default: ''
  },
  timestamp: {
    type: String,
    default: ''
  }
});

const emit = defineEmits(['reset']);

const defectClass = computed(() => {
  if (props.defectLevel === '一般缺陷') {
    return 'warning';
  } else if (props.defectLevel === '严重缺陷（一级）') {
    return 'critical-1';
  } else if (props.defectLevel === '危急缺陷') {
    return 'critical-2';
  }
  return '';
});
</script>

<style scoped>
.step {
  background: white;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
}

.result-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
}

.result-table th,
.result-table td {
  border: 1px solid #e0e0e0;
  padding: 0.8rem;
  text-align: left;
}

.result-table th {
  background-color: #f0f9f4;
  width: 30%;
}

.warning {
  background: #fff3cd;
}

.critical-1 {
  background: #ffe5d0;
}

.critical-2 {
  background: #f8d7da;
}

.btn {
  padding: 0.6rem 1.2rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 0.9rem;
  background: #42b983;
  color: white;
  margin-top: 1rem;
}
</style>