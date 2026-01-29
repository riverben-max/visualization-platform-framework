<template>
  <div class="left_bottom_inner">
    <dv-scroll-board v-if="pageflag" class="scroll-board" :config="scrollConfig" />
    <Reacquire v-else @click="getData" style="line-height: 200px">
      重新获取
    </Reacquire>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { currentGET } from '@/api/modules';

const pageflag = ref(true);
const scrollConfig = ref({
  header: ['编号', '物料及物料名', '仓库及仓库名', '当前库存及库存数', '状态'],
  data: [],
  index: false,
  columnWidth: [70, 170, 170, 160, 70],
  align: ['center', 'left', 'left', 'right', 'center'],
  rowNum: 5,
  headerBGC: '#193d8a',
  waitTime: 3000
});

const getStatusText = (item) => {

  const min = item.minInventory ?? null;
  const max = item.maxInventory ?? null;
  if (min !== null && item.totalInventory < min) return '不足';
  if (max !== null && item.totalInventory > max) return '溢出';
  return '正常';
};

const getStatusColor = (statusText) => {
  if (statusText === '不足') return '#00ff00';
  if (statusText === '溢出') return '#ff0000';
  return '#ffffff';
};

const getData = async () => {
  pageflag.value = true;
  try {
    const res = await currentGET('big3');
    console.log(res);
    const list = res.rows ||[];
    if (!Array.isArray(list) || list.length === 0) {
      pageflag.value = false;
      return;
    }

    const rows = list.map((item, index) => {
      const code = item.materialCode|| '';
      const materialName = item.materialName || '';
      const warehouseName = item.warehouseName || '';
      const currentStock = item.totalInventory ?? 0;
      const unit = item.materialUnit || '';

      const warehouseDisplay = warehouseName || '--';
      const stockDisplay = `${currentStock}${unit ? ' ' + unit : ''}`;

      const statusText = getStatusText(item);
      const statusColor = getStatusColor(statusText);

      return [
        code || String(index + 1).padStart(3, '0'),
        materialName,
        warehouseDisplay,
        stockDisplay,
        `<span style="color: ${statusColor};">${statusText}</span>`
      ];
    });

    scrollConfig.value = {
      ...scrollConfig.value,//是 展开运算符，意思是“把原来的配置全部拷贝一份”。
      data: rows
    };
  } catch (error) {
    console.error('库存预警数据获取失败:', error);
    pageflag.value = false;
  }
};

onMounted(() => {
  getData();
});
</script>

<style lang="scss" scoped>
.left_bottom_inner {
  width: 100%;
  height: 100%;
}

.scroll-board {
  width: 100%;
  height: 100%;
  margin-top: -10px;
}
</style>

