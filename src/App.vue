<template>
  <div id="app">
    <DefectAnalysisPage />
  </div>
</template>

<script setup>
import DefectAnalysisPage from './views/DefectAnalysisPage.vue';
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 0;
  width: 100%; /* 设置宽度为设备宽度 */
  height: 90vh; /* 设置高度为设备视口高度 */
  display: flex; /* 使用flex布局 */
  justify-content: center; /* 水平居中 */
  align-items: center; /* 垂直居中 */
}
</style>