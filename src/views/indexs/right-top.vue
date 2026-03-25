<template>
  <div class="right-top">
    <div id="rightTop" ref="chartRef" v-if="pageFalg"></div>
  </div>
</template>
<script setup>
import { nextTick, onBeforeUnmount, onMounted, ref, shallowRef } from 'vue';
import * as echarts from 'echarts'
import { currentGET } from '../../api/modules';

const chartRef = ref()
const chartIntance = shallowRef(null)
const option = shallowRef({})
const pageFalg = ref(false)

const currentData = ref({
  dateList: [],
  numList: [],
  numList2: [],

})

const initChart = () => {
  option.value = {
    tooltip: {
      trigger: 'axis',
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      boundaryGap: false,
      data: currentData.value.dateList
    },
    yAxis: [
      {
        type: 'value',
        name: '销售额',
        splitLine: { lineStyle: { color: 'rgba(255,255,255,0.08)' } }
      },
      {
        type: 'value',
        name: '合同额',
        splitLine: {
          show: false
        }
      }],
    series: [
      {
        name: '总销售额（万）',
        type: 'line',
        yAxisIndex: 0,
        smooth: true,
        data: currentData.value.numList
      },
      {
        name: '合同额（万）',
        type: 'line',
        yAxisIndex: 1,
        smooth: true,
        data: currentData.value.numList2
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
  const res = await currentGET('big4')
  if (!res || !res.data) return
  const data = res.data
  currentData.value.dateList = data.dateList
  currentData.value.numList = data.numList
  currentData.value.numList2 = data.numList2
  pageFalg.value = true
  initChart()
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
.right-top {
  height: 100%;
  width: 100%;
}

#rightTop {
  height: 100%;
  width: 100%;
}
</style>