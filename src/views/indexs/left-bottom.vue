<template>
  <div class="left-bottom-inner">
    <dv-scroll-board id="leftBottom" :config="config" />
  </div>
</template>
<script setup>
import { onMounted, ref } from 'vue';
import { currentGET } from '../../api/modules';
import { ElMessage } from 'element-plus';


const config = ref({
  header: ['编号', '物料', '仓库', '当前库存', '状态'],
  columnWidth: [70, 170, 170, 160, 70],
  data: [],
  align: ['center']
})

const getData = async () => {
  try {
    const res = await currentGET('big3')
    if (!res || !res.rows) return
    const rows = res.rows
    const list = rows.map((item) => {//!
      const id = item.id
      const materialName = item.materialName
      const warehouseName = item.warehouseName
      const num = item.totalInventory

      const max = item.maxInventory
      const min = item.minInventory
      const status = getStatus(max, min, num)
      const color = getStatusColor(status)
      const currentNum = num + item.materialUnit

      return [
        id,
        materialName,
        warehouseName,
        currentNum,
        `<span style="color:${color}">${status}</span>`//!!!
      ]

    })
    //!
    config.value = {
      ...config.value,
      data: list
    }
  } catch (e) {
    ElMessage('error')
  }
}

const getStatus = (max, min, num) => {
  if (num > max) return '溢出'
  if (num < min) return '不足'
  return '正常'
}

const getStatusColor = (s) => {
  if (s === '不足') return '#9fe6b8'
  if (s === '溢出') return '#fb7293'
  return '#ffffff'
}

onMounted(() => {
  getData()
})

</script>
<style scoped>
.left-bottom-inner {
  height: 100%;
  width: 100%;
}

#leftBottom {
  height: 95%;
  width: 100%;
}
</style>