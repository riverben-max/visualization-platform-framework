<template>
  <div class="count">
    <div v-if="pageFlag" ref="chartRef" class="chart-container"></div>
  </div>
</template>

<script setup name="custom">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { currentGET } from '../../api/modules'
import * as echarts from 'echarts'

const pageFlag = ref(true)
const chartRef = ref(null)
let chartInstance = null

const chartData = ref({
  category: [],
  barData: [],
  lineData: [],
  rateData: []
})

const handleResize = () => {
  chartInstance?.resize()
}

const initChart = () => {
  if (!chartRef.value) return

  if (!chartInstance) {
    chartInstance = echarts.init(chartRef.value)
    window.addEventListener('resize', handleResize)
  }

  const labelsAxis = chartData.value.category
  const soldQtyList = chartData.value.barData
  const planQtyList = chartData.value.lineData
  const ratePercentList = chartData.value.rateData

  const options = {
    tooltip: { trigger: 'axis', axisPointer: { type: 'cross' } },
    legend: { data: ['已销售', '计划销售', '完成率'] },
    grid: { left: 36, right: 46, top: 40, bottom: 26, containLabel: true },
    color: ['#4c9bfd', '#f6bd16', '#56b557'],
    xAxis: { type: 'category', data: labelsAxis },
    yAxis: [
      { type: 'value', name: '销售数量', splitLine: { lineStyle: { color: 'rgba(255,255,255,0.08)' } } },
      { type: 'value', name: '完成率(%)', min: 0, max: 100, splitLine: { show: false } }
    ],
    series: [
      { name: '已销售', type: 'bar', itemStyle: { borderRadius: [4, 4, 0, 0] }, data: soldQtyList },
      { name: '计划销售', type: 'bar', itemStyle: { borderRadius: [4, 4, 0, 0] }, data: planQtyList },
      { name: '完成率', type: 'line', yAxisIndex: 1, smooth: true, symbolSize: 6, lineStyle: { width: 2 }, data: ratePercentList }
    ]
  }

  chartInstance.setOption(options, true)
}

const getData = async () => {
  const res = await currentGET('big6')
  console.log(res);
  
  if (!res?.success) {
    pageFlag.value = false
    return
  }

  const data = res.data || {}
  chartData.value = {
    category: Array.isArray(data.category) ? data.category : [],
    barData: Array.isArray(data.barData) ? data.barData : [],
    lineData: Array.isArray(data.lineData) ? data.lineData : [],
    rateData: Array.isArray(data.rateData) ? data.rateData : []
  }

  initChart()
}

onMounted(() => {
  getData()
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
  chartInstance?.dispose()
  chartInstance = null
})
</script>


<style scoped>
.chart-container {
  width: 100%;
  height: 290px;
}
</style>