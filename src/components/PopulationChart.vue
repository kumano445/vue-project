<script setup lang="ts">
import { ref, defineExpose, onMounted } from "vue";
import Highcharts from "highcharts";
import HighchartsVue from "highcharts-vue"; // 追加

const chartContainer = ref<HTMLDivElement | null>(null);

const chartOptions = ref({
  chart: { type: "line" },
  title: { text: "" },
  xAxis: {
    categories: [],
    title: { text: "年" },
  },
  yAxis: {
    title: { text: "人口" },
  },
  series: [],
});

// チャートを更新する関数
const updateChart = (seriesData: { name: string; data: number[] }[]) => {
  console.log("Updating chart with data:", seriesData);
  chartOptions.value.series = seriesData;

  if (chartContainer.value) {
    Highcharts.chart(chartContainer.value, {
      ...chartOptions.value,
      series: seriesData,
    });
  }
};

// 他のコンポーネントから `updateChart` を呼び出せるようにする
defineExpose({ updateChart });

onMounted(() => {
  if (chartContainer.value) {
    Highcharts.chart(chartContainer.value, chartOptions.value);
  }
});
</script>

<template>
  <div>
    <div ref="chartContainer" style="height: 400px;"></div>
  </div>
</template>

