// PopulationChart.vue
<script setup lang="ts">
import { ref, defineExpose, onMounted } from "vue";
import Highcharts from "highcharts";

const chartContainer = ref<HTMLDivElement | null>(null);

const chartOptions = ref({
  chart: { type: "line" },
  title: { text: "" },
  xAxis: {
    categories: [], 
    title: { text: "年" },
    labels: {
      formatter: function () {
        return this.value; 
      },
    },
  },
  yAxis: {
    title: { text: "人口" },
  },
  series: [],
});

const updateChart = (chartData: { series: any[] }) => {
  chartOptions.value.series = chartData.series;
  if (chartContainer.value) {
    Highcharts.chart(chartContainer.value, chartOptions.value);
  }
};

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

