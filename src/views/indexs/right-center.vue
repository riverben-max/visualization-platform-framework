<template>
  <div class="right-center">
    <div class="box" ref="chartRef" v-if="pageFlag"></div>
  </div>
</template>

<script setup>
import { nextTick, onBeforeUnmount, onMounted, ref, shallowRef } from 'vue';
import * as echarts from 'echarts'
import { currentGET } from '../../api/modules';

const pageFlag = ref(false)
const chartRef = ref({})
const chartInstance = shallowRef(null)//！！！
const option = shallowRef({})

const currentData = ref({})

const initChart = () => {
  option.value = {
    grid: {
      left: 0,
      right: 60,
      top: 30,
      bottom: 10,
      containLabel: true
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow'
      }
    },
    xAxis: {
      name: '销售额',
      type: 'value',
      boundaryGap: [0, 0.01],
      axisLabel: {          // ← 添加这个配置
        color: '#fff'    // 设置 x 轴刻度标签的颜色（红色）
      }
    },
    yAxis: {
      name: '销售公司名称',
      type: 'category',
      data: currentData.value.names,
      axisLabel: {          // ← 添加这个配置
        color: '#fff'    // 设置 x 轴刻度标签的颜色（红色）
      }
    },
    series: [
      {
        name: '销量',
        type: 'bar',
        data: currentData.value.values,
        itemStyle: {
          color: (params) => {
            const colors = ['#5470C6', '#91CC75', '#FAC858', '#EE6666', '#73C0DE', '#3BA272']
            return colors[params.dataIndex % colors.length]
          }
        },
        label: {
          show: true,
          position: 'right',
          color: '#fff'
        }
      }
    ]
  };
}

const renderChart = () => {
  if (!chartRef.value) return
  if (!chartInstance.value) {
    chartInstance.value = echarts.init(chartRef.value)
  }
  chartInstance.value.setOption(option.value, true)
}

const getData = async () => {
  const res = await currentGET('big7')

  if (!res || !res.data) return
  const list = res.data.sort((a, b) => a.value - b.value)
  currentData.value = {//!!
    names: list.map(item => item.name),
    values: list.map(item => item.value)
  }

  initChart()
  pageFlag.value = true//!!
  nextTick(() => {
    renderChart()
  })
}

onMounted(() => {
  getData()
})
onBeforeUnmount(() => {
  chartInstance.value?.dispose()
})
</script>
<style scoped>
.right-center {
  height: 100%;
  width: 100%;
}

.box {
  height: 100%;
  width: 100%;
}
</style>