<script setup lang="ts">
import { ref, onMounted } from "vue";
import axios from "axios";
import PopulationChart from "./PopulationChart.vue";

interface Prefecture {
  prefCode: number;
  prefName: string;
}

const prefectures = ref<Prefecture[]>([]);
const selectedPrefectures = ref<number[]>([]);
const chartRef = ref<any>(null);
const selectedCategory = ref<string>("総人口");

// 都道府県一覧を取得
const fetchPrefectures = async () => {
  try {
    const response = await axios.get(
      "https://yumemi-frontend-engineer-codecheck-api.vercel.app/api/v1/prefectures",
      {
        headers: { "X-API-KEY": "8FzX5qLmN3wRtKjH7vCyP9bGdEaU4sYpT6cMfZnJ" },
      }
    );
    prefectures.value = response.data.result;
  } catch (error) {
    console.error("都道府県一覧の取得に失敗:", error);
  }
};


// 人口データを取得し、グラフを更新
const fetchPopulationData = async () => {
  const seriesData: any[] = [];
  for (const prefCode of selectedPrefectures.value) {
    try {
      const response = await axios.get(
      `https://yumemi-frontend-engineer-codecheck-api.vercel.app/api/v1/population/composition/perYear?prefCode=${prefCode}`,
      { headers: { "X-API-KEY": "8FzX5qLmN3wRtKjH7vCyP9bGdEaU4sYpT6cMfZnJ" } }
      );

      const populationData = response.data.result.data.find((item: any) => item.label === selectedCategory.value);
      seriesData.push({
        name: prefectures.value.find((p) => p.prefCode === prefCode)?.prefName,
        data: populationData.data.map((d: any) => d.value),
      });
    } catch (error) {
      console.error(`人口データ取得失敗: ${prefCode}`, error);
    }
  }
  chartRef.value?.updateChart(seriesData);
};

onMounted(fetchPrefectures);
</script>

<template>
  <div>
    <h2>都道府県一覧</h2>
    <div>
      <label v-for="pref in prefectures" :key="pref.prefCode">
        <input
          type="checkbox"
          :value="pref.prefCode"
          v-model="selectedPrefectures"
          @change="fetchPopulationData"
        />
        {{ pref.prefName }}
      </label>
    </div>

    <h3>人口種類を選択</h3>
    <select v-model="selectedCategory" @change="fetchPopulationData">
      <option value="総人口">総人口</option>
      <option value="年少人口">年少人口</option>
      <option value="生産年齢人口">生産年齢人口</option>
      <option value="老年人口">老年人口</option>
    </select>

    <!-- PopulationChart は PrefectureList 内で既に存在 -->
    <PopulationChart ref="chartRef" />
  </div>
</template>
