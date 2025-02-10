<script setup lang="ts">
import { ref, onMounted } from "vue";
import axios from "axios";
import PopulationChart from "./PopulationChart.vue";
import Highcharts from "highcharts";
import AccessibilityModule from "highcharts/modules/accessibility";

// Accessibilityモジュールを適用
if (typeof AccessibilityModule === "function") {
  AccessibilityModule(Highcharts);
} else {
  console.warn("Accessibility module is not a function.");
}



interface Prefecture {
  prefCode: number;
  prefName: string;
}

const prefectures = ref<Prefecture[]>([]);
const selectedPrefectures = ref<number[]>([]);
const chartRef = ref<any>(null);
const selectedCategory = ref<string>("総人口");

// 環境変数から API キーを取得
const RESAS_API_KEY = import.meta.env.VITE_RESAS_API_KEY;
console.log("API Key:", RESAS_API_KEY);

if (!RESAS_API_KEY) {
  console.error("RESAS APIキーが設定されていません。`.env.local` を確認してください。");
}


// 都道府県一覧を取得
const fetchPrefectures = async () => {
  try {
    const response = await axios.get(
      "https://yumemi-frontend-engineer-codecheck-api.vercel.app/api/v1/prefectures",
      {
        headers: { "X-API-KEY": RESAS_API_KEY },
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
  const years: string[] = []; 
  for (const prefCode of selectedPrefectures.value) {
    try {
      const response = await axios.get(
        `https://yumemi-frontend-engineer-codecheck-api.vercel.app/api/v1/population/composition/perYear?prefCode=${prefCode}`,
        { headers: { "X-API-KEY": RESAS_API_KEY }}
      );

      const populationData = response.data.result.data.find((item: any) => item.label === selectedCategory.value);
      const validData = populationData.data.map((d: any) => {
        return isNaN(d.value) ? 0 : d.value; // NaN を 0 に変換
      });
      const yearsForPref = populationData.data.map((d: any) => {
        const year = d.year; 

        return year >= 1900 ? year : year + 2000; // 例: 2000年以降はそのまま、1900年代は2000年代に補正
      });

      // 西暦の年度を更新
      years.push(...yearsForPref);

      seriesData.push({
        name: prefectures.value.find((p) => p.prefCode === prefCode)?.prefName,
        data: validData,
      });
    } catch (error) {
      console.error(`人口データ取得失敗: ${prefCode}`, error);
    }
  }
  chartRef.value?.updateChart({ series: seriesData });
};


onMounted(fetchPrefectures);
</script>

<template>
  <div class="container">
    <h2 class="title">都道府県一覧</h2>
    <div class="prefecture-list">
      <label v-for="pref in prefectures" :key="pref.prefCode" class="prefecture-item">
        <input
          type="checkbox"
          :value="pref.prefCode"
          v-model="selectedPrefectures"
          @change="fetchPopulationData"
        />
        {{ pref.prefName }}
      </label>
    </div>

    <h3 class="category-title">人口種類を選択</h3>
    <select v-model="selectedCategory" @change="fetchPopulationData" class="category-select">
      <option value="総人口">総人口</option>
      <option value="年少人口">年少人口</option>
      <option value="生産年齢人口">生産年齢人口</option>
      <option value="老年人口">老年人口</option>
    </select>

    <div class="chart-container">
      <PopulationChart ref="chartRef" />
    </div>
  </div>
</template>

<style scoped>
.container {
  padding: 20px;
  max-width: 800px;
  margin: 0 auto;
}

.title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
  text-align: center;
}

.prefecture-list {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  margin-bottom: 20px;
}

.prefecture-item {
  display: flex;
  align-items: center;
  font-size: 16px;
}

.category-title {
  font-size: 18px;
  margin-bottom: 10px;
}

.category-select {
  padding: 5px;
  font-size: 16px;
  margin-bottom: 20px;
}

.chart-container {
  height: 400px; /* 高さを調整 */
}
</style>
