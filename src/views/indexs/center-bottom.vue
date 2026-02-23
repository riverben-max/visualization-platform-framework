<template>
  <div class="count">
    <div v-if="pageflag" ref="chartRef" class="center_bottom_inner"></div>
   
  </div>
</template>
<script setup>

import { currentGET } from '@/api/modules';
import { ref } from 'vue';
import { onMounted, onBeforeUnmount} from 'vue'
import * as echarts from 'echarts'

const chartRef = ref(null)
const pageflag = ref(true)
let chartInstance = null
const chartData = ref({
  category: [],
  totalSales: [],
  contractSales: []
})
const getData = async () => {
  const res = await currentGET('big4')
  //console.log(res);
  if(!res?.success){
    pageflag.value = false
    return
  }
  const data = res.data || {}
  chartData.value = {
      category: data.dateList || [],
      totalSales: data.numList || [],
      contractSales: data.numList2 || []
    }

    // 3️⃣ 用新数据生成图表
    initChart()
}

const initChart = () => {
  if (chartInstance) {
    chartInstance.dispose()
  }

  // 创建新的图表实例
  chartInstance = echarts.init(chartRef.value)

  const labelsAxis = chartData.value.category
  const totalSalesList = chartData.value.totalSales
  const contractSalesList = chartData.value.contractSales

  const options = {
    tooltip:{
      trigger:'axis'
    },
    grid:{
      left:'3%',
      right:'4%',
      bottom:'3%',
      containLabel:true,
    },
    xAxis:{
      type:'category',boundaryGap:false,data:labelsAxis
    },
    yAxis:{
      type:'value',name:'销售额',splitLine: { lineStyle: { color: 'rgba(255,255,255,0.08)' } }
    },
    series:[
      {name:'总销售额',type:'line',smooth:'true',data:totalSalesList},
      {name:'合同额',type:'line',smooth:'true',data:contractSalesList}
    ]
    

  }  
  chartInstance.setOption(options)

  // 响应式调整大小
  const handleResize = () => {
    chartInstance?.resize()
  }
  window.addEventListener('resize', handleResize)

  
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
.center_bottom_inner{
  width: 100%;
  height: 290px;
}
</style>