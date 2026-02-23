<template>
  <div class="left_bottom_inner">
    <dv-scroll-board class="scroll-board" :config="scrollConfig" />
  </div>
</template>
<script setup>
import { ref } from 'vue';
import { currentGET } from '../../api/modules';
const scrollConfig = ref({
  header: ['编号', '物料及物料名称', '仓库及仓库名称', '当前库存及库存数量', '状态'],
  data: [],
  columnWidth: [70, 170, 170, 160, 70],
  align: ['center'],
  headerBGC: '#193d8a'
})

const getData = async () => {
  try {
    const res = await currentGET('big3')
    console.log(res);
    const rows = res.rows || [];
    if (!rows || rows.length === 0) {
      console.log('没数据');
      return;
    }
    const list = rows.map((rows) => {
      const id = rows.id;
      const materialName = rows.materialName;
      const warehouseName = rows.warehouseName;
      const totalInventory = rows.totalInventory;
      const unit = rows.materialUnit;
      const stockDisplay = `${totalInventory}${unit ? ' ' + unit : ''}`

      const status = getStatusText(rows);
      const statusColor = getStatusTextColor(status);

      return [
        id || String(index + 1).padStart(3, '0'),
        materialName,
        warehouseName,
        stockDisplay,
        `<span style="color:${statusColor}">${status}</span>`
      ];
    })

    scrollConfig.value ={
      //把原来 scrollConfig 里的所有属性拷贝过来，
      ...scrollConfig.value,
      data:list
    }
  }
  catch (error) {
    console.error('库存预警数据获取失败:', error);
    pageflag.value = false;
  }
}

function getStatusText(rows) {
  const min = rows.minInventory ?? null;
  const max = rows.maxInventory ?? null;
  if (min !== null && rows.totalInventory < min) return '不足';
  if (max !== null && rows.totalInventory > max) return '溢出';
  return '正常';
}

function getStatusTextColor(statusText) {
  if (statusText === '不足') return '#00ff00';
  if (statusText === '溢出') return '#ff0000';
  return '#ffffff';
}

onMounted(() => {
  getData();
});
</script>
<style lang="scss" scoped>
.left_bottom_inner {
 width: 100%;
 height: 100%;
 background: rgba(255, 0, 0, 0.2);
}

.scroll-board {
 width: 100%;
 height: 100%;
 margin-top: -10px;
 background-color: aqua;
}
</style>