<template>
  <div class="left-center">
    <div v-if="pageFlag" ref="chartRef" id="leftCenter"></div>
    <Reacquire v-else @click="getData">重新获取</Reacquire>
  </div>
</template>

<script setup>
import { nextTick, onBeforeUnmount, onMounted, reactive, ref, shallowRef } from 'vue';
import { currentGET } from '../../api/modules';
import * as echarts from 'echarts'

const pageFlag = ref(true)
const options = shallowRef({})
const chartRef = ref(null)
const chartInstance = shallowRef(null)
const timer = ref(null)

const data = reactive({
  totalPowerConsume: 0,
  totalOfficeWaterConsume: 0,
  totalWaterConsume: 0,
  totalOfficePowerConsume: 0
})

const initChart = () => {
  const total = data.totalOfficePowerConsume + data.totalOfficeWaterConsume + data.totalPowerConsume + data.totalWaterConsume
  options.value = {
    tooltip: {
      trigger: 'item'
    },
    title: {
      text: `{value|${total}}\n{num|总数}`,//!!!
      top: 'center',
      left: 'center',//!
      textStyle: {//!
        rich: {
          value: {
            color: '#fff',
          },
          num: {
            color: '#fff'
          }
        }
      }
    },
    series: [
      {
        name: '能耗占比',
        type: 'pie',
        radius: ['35%', '75%'],
        avoidLabelOverlap: false,
        itemStyle: {
          borderRadius: 5,
          borderColor: '#fff',
          borderWidth: 1
        },
        label: {
          show: true,
          formatter: '{b|{b}} \n {c|{c}}  {per|{d}%}',
          rich: {
            b: {
              color: '#fff',
              fontSize: 14
            },
            c: {
              color: '#aaa'
            },
            per: {
              color: '#aaa'
            }

          }
        },
        data: [
          { value: data.totalOfficePowerConsume, name: '办公耗电量' },
          { value: data.totalOfficeWaterConsume, name: '办公用水量' },
          { value: data.totalPowerConsume, name: '生产耗电量' },
          { value: data.totalWaterConsume, name: '生产用水量' },
        ]
      }
    ]
  };
}
const renderChart = () => {
  if (!chartRef.value) return
  if (!chartInstance.value) {
    chartInstance.value = echarts.init(chartRef.value)
  }
  chartInstance.value.setOption(options.value, true)  //!
}
const getData = async () => {
  try {
    const res = await currentGET('big1')
    if (!res || !res.data[0]) {
      pageFlag.value = false
      clearInterval(timer.value)  // ✅ 停止定时器
      timer.value = null
      return
    }
    pageFlag.value = true
    const currentData = res.data[0]
    data.totalOfficePowerConsume = currentData.totalOfficePowerConsume
    data.totalOfficeWaterConsume = currentData.totalOfficeWaterConsume
    data.totalPowerConsume = currentData.totalPowerConsume
    data.totalWaterConsume = currentData.totalWaterConsume

    initChart()
    nextTick(() => {
      renderChart()
      swipter()
    })
  } catch (e) {//!
    console.error('错误数据', e)
    pageFlag.value = false
    clearInterval(timer.value)//!
    timer.value = null
  }

}
const swipter = () => {
  if (timer.value) return
  timer.value = setInterval(getData, 3000)
}
onMounted(() => {
  getData()
})
onBeforeUnmount(() => {
  clearInterval(timer.value)
  chartInstance.value?.dispose()
  chartInstance.value = null
})

</script>
<style scoped>
.left-center {
  width: 100%;
  height: 100%;
}

#leftCenter {
  width: 100%;
  height: 100%;
}
</style>
