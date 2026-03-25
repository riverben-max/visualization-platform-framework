<template>
  <div class="center-top">
    <div class="box">
      <div v-for="item in salesOverview" :key="item.key">
        <div>{{ item.title }}</div>
        <dv-digital-flop :config="getFlopConfig(item.value)" style="height: 35px;" />
      </div>
    </div>
    <div class="box">
      <div v-for="item in emissionOverview" :key="item.key">
        <div>{{ item.title }}</div>
        <dv-digital-flop :config="getFlopConfig(item.value)" style=" height: 35px;" />
      </div>
    </div>

    <div>
      <dv-scroll-board :config="config" style="width:750px;height:420px" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { currentGET } from '@/api/modules/index.js';

// 初始状态为 0
const salesStats = ref({
  year: 0,
  month: 0,
  day: 0
});

const emissionStats = ref({
  year: 0,
  month: 0,
  day: 0
});

const salesOverview = computed(() => ([
  { key: 'sale-year', title: '今年累计销售额', value: salesStats.value.year },
  { key: 'sale-month', title: '本月累计销售额', value: salesStats.value.month },
  { key: 'sale-day', title: '今日累计销售额', value: salesStats.value.day }
]));

const emissionOverview = computed(() => ([
  { key: 'emission-year', title: '本年累计碳排放', value: emissionStats.value.year },
  { key: 'emission-month', title: '本月累计碳排放', value: emissionStats.value.month },
  { key: 'emission-day', title: '今日累计碳排放', value: emissionStats.value.day }
]));

const getFlopConfig = (val) => ({
  number: [Number(val) || 0],
  content: '{nt}',
  style: {
    fontSize: 22,
    fill: '#67e0e3',
    fontWeight: 'bold'
  }
});

const config = ref({
  header: ['设备', '生产数量', '碳排放'],
  data: [],
  index: true,
  indexHeader: '编号',
  columnWidth: [170, 200, 200, 200],
});


const getData = async () => {
  // 1. 恢复上半部分的数据总览获取逻辑（big12, big10, big9, big8）
  const sales = await currentGET('big12')
  const year = await currentGET('big10')
  const month = await currentGET('big9')
  const day = await currentGET('big8')

  if (sales.data) {
    salesStats.value.year = sales.data.yearTotal
    salesStats.value.day = sales.data.todayTotal
  }

  if (year.rows[0]) emissionStats.value.year = year.rows[0].totalCarbonEmission
  if (month.rows[0]) emissionStats.value.month = month.rows[0].totalCarbonEmission
  if (day.rows[0]) emissionStats.value.day = day.rows[0].totalCarbonEmission

  // 2. 修复下半部分的排行榜获取逻辑 (big11)
  const res = await currentGET('big11');
  if (!res || !Array.isArray(res.rows)) return;

  const rows = res.rows.sort((a, b) => b.carbonEmission - a.carbonEmission).slice(0, 8);//!!!
  const list = rows.map((item) => {
    const equipmentName = item.equipmentName
    const productQuantity = item.productQuantity
    const carbonEmission = item.carbonEmission
    return [
      equipmentName,
      productQuantity,
      carbonEmission
    ]
  })
  config.value = {
    ...config.value,
    data: list
  }

};

onMounted(() => {
  getData();
});

</script>

<style lang="scss" scoped>
.center-top {
  height: 100%;
  width: 100%;
}

.box {
  display: flex;
  justify-content: space-around;
  margin-bottom: 10px;
}
</style>
