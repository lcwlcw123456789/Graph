<template>
  <div id="app">
    <div class="chart-wrapper">
      <h1>Simple Bar Chart: From Liangcw</h1>
      <BarChart />
    </div>
  </div>
</template>

<script>
import BarChart from "./components/BarChart.vue";

export default {
  components: {
    BarChart,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh; /* 页面高度 */
  width: 100vw;  /* 页面宽度 */
  margin: 1;
}

.chart-wrapper {
  text-align: center;
}

h1 {
  font-size: 20px; /* 调整字体大小 */
  font-weight: bold;
  margin-bottom: 10px; /* 减小标题与图表的间距 */
}
</style>
