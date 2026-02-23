<template>
  <div class="right-center-inner">
    <div v-if="pageFlag" ref="chartRef" class="chart-box"></div>
    <div v-else class="empty-text">暂无数据</div>
  </div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref, nextTick } from 'vue';
import * as echarts from 'echarts';
import { currentGET } from '@/api/modules';

const pageFlag = ref(true);
const chartRef = ref(null);
let myChart = null;

const chartData = ref({
  names: [],
  values: []
});

const toNumber = (value) => {
  const num = Number(value);
  return Number.isFinite(num) ? num : 0;
};

const formatNumber = (value) => {
  const num = Number(value);
  if (!Number.isFinite(num)) return '--';
  return num.toLocaleString();
};

const initChart = () => {
  if (!chartRef.value) return;
  if (!chartRef.value.clientWidth || !chartRef.value.clientHeight) return;

  if (!myChart) {
    myChart = echarts.init(chartRef.value);
  }

  const colors = [
    '#3ba1ff',
    '#4fd97f',
    '#f6bd16',
    '#f58b3c',
    '#8b7bff',
    '#2ec7c9',
    '#ff6b6b',
    '#7bd8ff'
  ];

  myChart.setOption(
    {
      grid: {
        left: 100,
        right: 50,
        top: 20,
        bottom: 24,
        containLabel: true
      },
      tooltip: {
        trigger: 'axis',
        axisPointer: { type: 'shadow' },
        formatter: (params) => {
          const item = params?.[0];
          if (!item) return '';
          return `${item.name}<br/>销售额：${formatNumber(item.value)}`;
        }
      },
      xAxis: {
        name: '销售额',
        type: 'value',
        axisLine: { lineStyle: { color: 'rgba(255,255,255,0.3)' } },
        axisLabel: { color: '#cfe6ff' },
        splitLine: { lineStyle: { color: 'rgba(255,255,255,0.08)' } }
      },
      yAxis: {
        type: 'category',
        inverse: true,
        data: chartData.value.names,
        axisTick: { show: false },
        axisLine: { lineStyle: { color: 'rgba(255,255,255,0.2)' } },
        axisLabel: { color: '#fff' }
      },
      series: [
        {
          name: '销售排名',
          type: 'bar',
          barWidth: 14,
          data: chartData.value.values,
          itemStyle: {
            color: (params) => colors[params.dataIndex % colors.length],
            borderRadius: [0, 4, 4, 0]
          },
          label: {
            show: true,
            position: 'right',
            color: '#fff',
            formatter: (params) => formatNumber(params.value)
          }
        }
      ]
    },
    true
  );
};

const renderChartWhenReady = async () => {
  await nextTick();
  requestAnimationFrame(() => {
    initChart();
    myChart?.resize();
  });
};

const getData = async () => {
  try {
    const res = await currentGET('big7');
    console.log('big7 res:', res);

    if (!res?.success || !Array.isArray(res.data)) {
      pageFlag.value = false;
      return;
    }

    const list = res.data
      .map((item) => ({
        name: item?.name || '--',
        value: toNumber(item?.value)
      }))
      .sort((a, b) => b.value - a.value)
      .slice(0, 8);

    if (!list.length) {
      pageFlag.value = false;
      return;
    }

    chartData.value = {
      names: list.map((item) => item.name),
      values: list.map((item) => item.value)
    };

    pageFlag.value = true;
    await renderChartWhenReady();
  } catch (error) {
    console.error('sale ranking fetch error:', error);
    pageFlag.value = false;
  }
};

const handleResize = () => {
  myChart?.resize();
};

onMounted(() => {
  getData();
  window.addEventListener('resize', handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize);
  if (myChart) {
    myChart.dispose();
    myChart = null;
  }
});
</script>

<style lang="scss" scoped>
.right-center-inner {
  width: 100%;
  height: 100%;
}

.chart-box {
  width: 100%;
  height: 100%;
}

.empty-text {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #8aa6c7;
  font-size: 14px;
}
</style>
