<template>
  <div class="center-bottom">
    <div ref="chartRef" id="centerBottom" v-if="pageFlag"></div>
  </div>
</template>
<script setup>
import { nextTick, onBeforeUnmount, onMounted, ref, shallowRef } from 'vue';
import * as echarts from 'echarts'
import { currentGET } from '../../api/modules';

const pageFlag = ref(false)
const chartRef = ref()
const chartIntance = shallowRef(null)
const option = shallowRef({})
const currentData = ref({
  barData: [],
  category: [],
  lineData: [],
  rateData: [],
})

const initChart = () => {
  option.value = {

    tooltip: {
      trigger: 'axis'
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true
    },

    xAxis: {
      type: 'category',
      boundaryGap: true,
      data: currentData.value.category
    },
    yAxis: [{
      name: '销售数量',
      splitLine: {
        show: false
      },
      type: 'value'
    },
    {
      name: '完成率',
      type: 'value'
    }
    ],
    series: [
      {
        name: '完成率',
        yAxisIndex: '1',
        type: 'line',
        data: currentData.value.rateData
      },
      {
        name: '已销售',
        type: 'bar',
        data: currentData.value.barData
      },
      {
        name: '计划销售',
        type: 'bar',
        data: currentData.value.lineData
      },

    ]
  };
}
const renderChart = () => {
  if (!chartRef.value) return
  if (!chartIntance.value) {
    chartIntance.value = echarts.init(chartRef.value)
  }
  chartIntance.value.setOption(option.value, true)
}
const getData = async () => {
  const res = await currentGET('big6')
  if (!res || !res.data) return
  const data = res.data
  currentData.value.barData = data.barData
  currentData.value.category = data.category
  currentData.value.lineData = data.lineData
  currentData.value.rateData = data.rateData
  initChart()
  pageFlag.value = true
  nextTick(() => {
    renderChart()
  })
}
onMounted(() => {
  getData()
})
onBeforeUnmount(() => {
  chartIntance.value?.dispose()
  chartIntance.value = null
})
</script>
<style scoped>
.center-bottom {
  height: 100%;
  width: 100%;
}

#centerBottom {
  height: 100%;
  width: 100%;
}
</style>