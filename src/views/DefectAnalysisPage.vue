<template>
  <div class="container">
    <div class="header">
      <h1>设备缺陷分析系统</h1>
    </div>
    <!-- 为内容添加一个容器，设置顶部外边距以避免被标题覆盖 -->
    <div class="content scrollable" ref="contentRef">
      <DeviceSelection
        :devices="devices"
        :selectedDevice="selectedDevice"
        :error="error"
        @device-selected="selectDevice"
      />
      <TemperatureInput
        v-if="step === 2"
        :T0="T0"
        :T1="T1"
        :T2="T2"
        :error="error"
        @go-back="handleGoBack"
        @calculate="calculateDelta"
        @update:T0="T0 = $event"
        @update:T1="T1 = $event"
        @update:T2="T2 = $event"
      />
      <ResultDisplay
        v-if="step === 3"
        :delta="delta"
        :defectLevel="defectLevel"
        :defectConditions="defectConditions"
        :T0="T0"
        :T1="T1"
        :T2="T2"
        :device="devices[selectedDevice]"
        :timestamp="timestamp"
        @reset="reset"
      />
      <div v-if="measurementHistory.length > 0">
        <button @click="toggleHistory" class="btn secondary">
          {{ isHistoryExpanded ? '收起测量履历' : '展开测量履历' }}
        </button>
        <div v-if="isHistoryExpanded">
          <input
            v-model="searchQuery"
            type="text"
            placeholder="搜索履历..."
            class="search-input"
          />
          <div v-for="(group, device) in filteredGroupedHistory" :key="device">
            <h3>{{ device }}</h3>
            <div class="history-cards" v-if="isMobile">
              <div
                v-for="history in paginatedGroupedHistory[device]"
                :key="history.timestamp"
                :class="getDefectClass(history.defectLevel)"
                class="history-card"
              >
                <p><strong>测量日期和时间:</strong> {{ history.timestamp }}</p>
                <p><strong>温度数值 (T0, T1, T2):</strong> ({{ history.T0 }}℃, {{ history.T1 }}℃, {{ history.T2 }}℃)</p>
                <p><strong>δ值:</strong> {{ history.delta }}%</p>
                <p><strong>缺陷等级:</strong> {{ history.defectLevel }}</p>
              </div>
            </div>
            <table class="history-table" v-else>
              <thead>
                <tr>
                  <th>测量日期和时间</th>
                  <th>温度数值 (T0, T1, T2)</th>
                  <th>δ值</th>
                  <th>缺陷等级</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="history in paginatedGroupedHistory[device]"
                  :key="history.timestamp"
                  :class="getDefectClass(history.defectLevel)"
                >
                  <td>{{ history.timestamp }}</td>
                  <td>{{ `(${history.T0}℃, ${history.T1}℃, ${history.T2}℃)` }}</td>
                  <td>{{ history.delta }}%</td>
                  <td>{{ history.defectLevel }}</td>
                </tr>
              </tbody>
            </table>
            <div v-if="paginatedGroupedHistory[device].length > itemsPerPage">
              <button
                @click="prevPage(device)"
                :disabled="currentPage[device] === 1"
                class="btn page-btn"
              >
                上一页
              </button>
              <span>{{ currentPage[device] }} / {{ totalPages[device] }}</span>
              <button
                @click="nextPage(device)"
                :disabled="currentPage[device] === totalPages[device]"
                class="btn page-btn"
              >
                下一页
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import DeviceSelection from '../components/DeviceSelection.vue';
import TemperatureInput from '../components/TemperatureInput.vue';
import ResultDisplay from '../components/ResultDisplay.vue';

const DEFECT_RULES = [
  { // 设备1
    level3: { delta: 95, temp: 80 }
  },
  { // 设备2 - 5
    level3: { delta: 95, temp: 90 }
  },
  { // 设备6 - 9
    level3: { delta: 95, temp: 55 }
  }
];

const step = ref(1);
const selectedDevice = ref(null);
const T0 = ref(null);
const T1 = ref(null);
const T2 = ref(null);
const delta = ref(null);
const error = ref(null);
const devices = [
  "电器设备与金属部件的连接接头和线夹",
  "金属部件与金属部件的连接接头和线夹",
  "金属导线接头和线夹",
  "隔离开关转头",
  "隔离开关刀口",
  "断路器动静触头",
  "断路器动静触头中间触头",
  "套管柱头",
  "电容器熔丝",
  "变压器箱体"
];
const defectLevel = ref('');
const defectConditions = ref('');
const timestamp = ref('');
const measurementHistory = ref([]);
const isHistoryExpanded = ref(false);
const searchQuery = ref('');
const itemsPerPage = ref(5);
const currentPage = ref({});
const totalPages = ref({});
const isMobile = ref(window.innerWidth <= 768);

const handleResize = () => {
  isMobile.value = window.innerWidth <= 768;
};

window.addEventListener('resize', handleResize);

const contentRef = ref(null);
const selectDevice = (index) => {
  selectedDevice.value = index;
  error.value = null;
  step.value = 2;

  // 点击计算后滚动到最底端
  if (contentRef.value) {
    contentRef.value.scrollTop = contentRef.value.scrollHeight;
  }
};

const validateInputs = () => {
  if ([T0.value, T1.value, T2.value].some(v => v === null || isNaN(v))) {
    error.value = "请输入有效的温度值，例如 25。";
    return false;
  }
  if (T1.value === T0.value) {
    error.value = "T1 不能等于 T0，请重新输入。";
    return false;
  }
  error.value = null;
  return true;
};

const calculateDelta = () => {
  if (!validateInputs()) return;

  const deltaValue = ((T1.value - T2.value) / (T1.value - T0.value) * 100);
  delta.value = deltaValue.toFixed(2);
  timestamp.value = new Date().toLocaleString();
  determineDefect();
  step.value = 3;
  measurementHistory.value.push({
    timestamp: timestamp.value,
    device: devices[selectedDevice.value],
    T0: T0.value,
    T1: T1.value,
    T2: T2.value,
    delta: delta.value,
    defectLevel: defectLevel.value
  });
  // 点击计算后滚动到最底端
  if (contentRef.value) {
    contentRef.value.scrollTop = contentRef.value.scrollHeight;
  }
};

const determineDefect = () => {
  const deviceType = getDeviceType();
  const deltaValue = parseFloat(delta.value);

  let conditions = '';
  let level = '';

  switch (deviceType) {
    case 0:
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
    case 6:
    case 7:
    case 8:
      if (deltaValue >= 95) {
        level = '危急缺陷';
        conditions = 'δ ≥ 95%';
      } else if (deltaValue >= 80) {
        level = '严重缺陷';
        conditions = '95% > δ ≥ 80%';
      } else if (deltaValue >= 35) {
        level = '一般缺陷';
        conditions = '80% > δ ≥ 35%';
      } else {
        level = '合格';
        conditions = 'δ < 35%';
      }
      break;
    case 9:
      if (deltaValue >= 35) {
        level = '一般缺陷';
        conditions = 'δ ≥ 35%';
      } else {
        level = '合格';
        conditions = 'δ < 35%';
      }
      break;
  }

  defectLevel.value = level;
  defectConditions.value = conditions;
};

/* const getDeviceType = () => {
  if (selectedDevice.value === 0) return 0;
  if (selectedDevice.value >= 1 && selectedDevice.value <= 4) return 1;
  return 2;
}; */

const getDeviceType = () => {
  return selectedDevice.value;
};

const reset = () => {
  step.value = 1;
  selectedDevice.value = null;
  T0.value = null;
  T1.value = null;
  T2.value = null;
  delta.value = null;
  error.value = null;
  defectLevel.value = '';
  defectConditions.value = '';
};

const handleGoBack = () => {
  step.value = 1;
  error.value = null;
};

const groupedHistory = computed(() => {
  const groups = {};
  const sortedHistory = [...measurementHistory.value].sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));
  sortedHistory.forEach((history) => {
    const device = history.device;
    if (!groups[device]) {
      groups[device] = [];
    }
    groups[device].push(history);
  });
  return groups;
});

const filteredGroupedHistory = computed(() => {
  const query = searchQuery.value.toLowerCase();
  const filtered = {};
  for (const [device, histories] of Object.entries(groupedHistory.value)) {
    const filteredHistories = histories.filter((history) => {
      return (
        history.timestamp.toLowerCase().includes(query) ||
        history.device.toLowerCase().includes(query) ||
        history.delta.toString().includes(query) ||
        history.defectLevel.toLowerCase().includes(query) ||
        `${history.T0} ${history.T1} ${history.T2}`.includes(query)
      );
    });
    if (filteredHistories.length > 0) {
      filtered[device] = filteredHistories;
    }
  }
  return filtered;
});

const paginatedGroupedHistory = computed(() => {
  const paginated = {};
  for (const [device, histories] of Object.entries(filteredGroupedHistory.value)) {
    const startIndex = (currentPage.value[device] - 1) * itemsPerPage.value;
    const endIndex = startIndex + itemsPerPage.value;
    paginated[device] = histories.slice(startIndex, endIndex);
    totalPages.value[device] = Math.ceil(histories.length / itemsPerPage.value);
    if (!currentPage.value[device]) {
      currentPage.value[device] = 1;
    }
  }
  return paginated;
});

const getDefectClass = (level) => {
  if (level === '一般缺陷') {
    return 'warning';
  } else if (level === '严重缺陷') {
    return 'critical-1';
  } else if (level === '危急缺陷') {
    return 'critical-2';
  }
  return 'normal';
};

const toggleHistory = () => {
  isHistoryExpanded.value = !isHistoryExpanded.value;
};

const prevPage = (device) => {
  if (currentPage.value[device] > 1) {
    currentPage.value[device]--;
  }
};

const nextPage = (device) => {
  if (currentPage.value[device] < totalPages.value[device]) {
    currentPage.value[device]++;
  }
};
</script>

<style scoped>
/* 确保 html 和 body 没有默认的边距 */
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  overflow-x: hidden; /* 防止出现水平滚动条 */
}

.container {
  min-height: 65vh;
  width: 100%; /* 设置宽度为 100% */
  background: #c4e4dc;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
  margin: 0; /* 去除外边距 */
  padding: 0; /* 去除内边距 */
}

.header {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  background: #70c4bf;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  z-index: 1;
  text-align: center;
  padding: 1rem;
  position: sticky;
}

.content {
  margin-top: 10px; /* 顶部外边距，避免内容被标题覆盖 */
  padding: 0.1rem;
  width: 100%;
  box-sizing: border-box;
  padding-top: calc(1rem + 头部的高度); /* 确保内容不被头部覆盖 */
  max-height: calc(100vh - 100px); /* 设置最大高度，减去头部高度 */
  overflow-y: auto; /* 启用垂直滚动 */
}

.scrollable {
  max-height: calc(100vh - 100px); /* 可以根据实际情况调整或者去掉 */
  overflow-y: auto;
}

/* 确保子组件宽度与容器一致 */
.DeviceSelection,
.TemperatureInput,
.ResultDisplay,
.history-section {
  width: 100%;
  box-sizing: border-box;
}

.history-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 1.5rem;
}

.history-table th,
.history-table td {
  border: 1px solid #e0e0e0;
  padding: 0.8rem;
  text-align: left;
}

.history-table th {
  background-color: #f0f9f4;
}

.normal {
  background: #d0f3b8;
}

.warning {
  background: #e7d9a8;
}

.critical-1 {
  background: #eec8aa;
}

.critical-2 {
  background: #d88080;
}

.btn {
  padding: 0.6rem 1.2rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 0.9rem;
  margin-top: 1rem;
}

.secondary {
  background: #e0e0e0;
}

.search-input {
  padding: 0.6rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-bottom: 1rem;
  width: 100%;
}

.page-btn {
  margin: 0 0.5rem;
}

.TemperatureInput input {
  width: 70%;
}

.history-card {
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  padding: 1rem;
  margin-bottom: 1rem;
}

@media (max-width: 768px) {
  .container {
    padding: 0.1rem;
  }

  .header {
    padding: 0.5rem;
  }

  .content {
    margin-top: 10px;
  }

  .history-table th,
  .history-table td {
    padding: 0.4rem;
    font-size: 0.8rem;
  }

  .btn {
    padding: 0.1rem 0.2rem;
    font-size: 0.8rem;
  }
}
</style>