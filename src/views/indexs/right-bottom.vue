<template>
  <div class="right-bottom">
    <dv-scroll-board :config="config" style="width:100%;height:95%" />
  </div>
</template>
<script setup>
import { onMounted, ref } from 'vue';
import { currentGET } from '../../api/modules';

const config = ref({
  header: ['编号', '产品', '型号', '规格', '单位', '库存', '已生成', '代生产', '碳排放'],
  data: [],
  columnWidth: [50, 70, 70, 70, 50, 50, 40, 40, 60],
  align: ['center'],
})

const getData = async () => {
  const res = await currentGET('big5')

  if (!res || !Array.isArray(res.rows)) return
  const rows = res.rows
  const list = rows.map((item) => {
    const materialId = item.materialId
    const materialName = item.materialName
    const materialModel = item.materialModel
    const materialSpecification = item.materialSpecification
    const materialUnit = item.materialUnit
    const totalInventory = item.totalInventory
    const productQuantity = item.productQuantity
    const requireQuantity = item.requireQuantity
    const carbonEmission = item.carbonEmission

    return [
      materialId,
      materialName,
      materialModel,
      materialSpecification,
      materialUnit,
      totalInventory,
      productQuantity,
      requireQuantity,
      carbonEmission
    ]
  })
  config.value = {
    ...config.value,
    data: list
  }
}

onMounted(() => {
  getData()
})
</script>
<style scoped>
.right-bottom {
  height: 100%;
  width: 100%;
}
</style>