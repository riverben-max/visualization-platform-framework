<template>
  <div class="center_top_inner">
    <div class="overview">
      <div class="overview-row">
        <div class="overview-card" v-for="item in salesOverview" :key="item.key">
          <div class="overview-title">{{ item.title }}</div>
          <div class="overview-value">{{ item.value }}</div>
        </div>
      </div>
      <div class="overview-row">
        <div class="overview-card" v-for="item in emissionOverview" :key="item.key">
          <div class="overview-title">{{ item.title }}</div>
          <div class="overview-value">{{ item.value }}</div>
        </div>
      </div>
    </div>

    <div class="rank-table">
      <div class="table-header">
        <span>编号</span>
        <span>设备</span>
        <span>生产数量</span>
        <span>碳排放</span>
      </div>
      <div class="table-body">
        <div class="table-row" v-for="row in displayRows" :key="row.key">
          <span class="col-index">{{ row.index }}</span>
          <span class="col-device">{{ row.device }}</span>
          <span class="col-qty">{{ row.production }}</span>
          <span class="col-emission">{{ row.emission }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { currentGET } from '@/api/modules';

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

const rankList = ref([]);

const salesOverview = computed(() => ([
  { key: 'sale-year', title: '今年累计销售额', value: formatNumber(salesStats.value.year) },
  { key: 'sale-month', title: '本月累计销售额', value: formatNumber(salesStats.value.month) },
  { key: 'sale-day', title: '今日累计销售额', value: formatNumber(salesStats.value.day) }
]));

const emissionOverview = computed(() => ([
  { key: 'emission-year', title: '本年累计碳排放', value: formatNumber(emissionStats.value.year) },
  { key: 'emission-month', title: '本月累计碳排放', value: formatNumber(emissionStats.value.month) },
  { key: 'emission-day', title: '今日累计碳排放', value: formatNumber(emissionStats.value.day) }
]));

const displayRows = computed(() => {
  const rows = [...rankList.value].slice(0, 8);
  while (rows.length < 8) {
    rows.push({});
  }
  return rows.map((row, index) => ({
    key: row.key || row.id || `${index + 1}`,
    index: index + 1,
    device: row.device || '--',
    production: row.productionDisplay || '--',
    emission: row.emissionDisplay || '--'
  }));
});

const getFirstRecord = (res) => {
  if (!res) return {};
  if (Array.isArray(res.data)) return res.data[0] || {};
  if (Array.isArray(res.rows)) return res.rows[0] || {};
  if (Array.isArray(res.data?.rows)) return res.data.rows[0] || {};
  if (res.data && typeof res.data === 'object') return res.data;
  return res || {};
};

const pickValue = (record, keys) => {
  for (const key of keys) {
    if (record && record[key] !== undefined && record[key] !== null) {
      return record[key];
    }
  }
  return 0;
};

const toNumber = (value) => {
  const num = Number(value);
  return Number.isFinite(num) ? num : 0;
};

const formatNumber = (value) => {
  const num = Number(value);
  if (!Number.isFinite(num)) return '--';
  return num.toLocaleString();
};

const buildRankList = (list) => {
  return list.map((item, index) => {
    const device = pickValue(item, [
      'equipmentName',
      'deviceName',
      'equipment',
      'name',
      'device'
    ]);
    const production = pickValue(item, [
      'productionNum',
      'productionCount',
      'productNum',
      'productCount',
      'productQuantity',
      'output',
      'quantity'
    ]);
    const emission = pickValue(item, [
      'totalCarbonEmission',
      'carbonEmission',
      'totalEmission',
      'emission',
      'carbon'
    ]);
    return {
      key: item.id || `${index + 1}`,
      device: device || '--',
      productionValue: toNumber(production),
      productionDisplay: formatNumber(production),
      emissionValue: toNumber(emission),
      emissionDisplay: formatNumber(emission)
    };
  });
};

const getOverviewData = async () => {
  try {
    const [saleRes, yearRes, monthRes, dayRes, rankRes] = await Promise.all([
      currentGET('big12'),
      currentGET('big10'),
      currentGET('big9'),
      currentGET('big8'),
      // currentGET('big11')
    ]);

    const saleRecord = getFirstRecord(saleRes);
    salesStats.value = {
      year: toNumber(pickValue(saleRecord, [
        'yearSaleAmount',
        'yearSalesAmount',
        'yearSale',
        'yearSales',
        'yearTotalSale',
        'totalYearSale',
        'totalYearSales'
      ])),
      month: toNumber(pickValue(saleRecord, [
        'monthSaleAmount',
        'monthSalesAmount',
        'monthSale',
        'monthSales',
        'monthTotalSale',
        'totalMonthSale',
        'totalMonthSales'
      ])),
      day: toNumber(pickValue(saleRecord, [
        'daySaleAmount',
        'daySalesAmount',
        'daySale',
        'daySales',
        'dayTotalSale',
        'totalDaySale',
        'totalDaySales'
      ]))
    };

    emissionStats.value = {
      year: toNumber(pickValue(getFirstRecord(yearRes), [
        'totalCarbonEmission',
        'carbonEmission',
        'totalEmission',
        'emission'
      ])),
      month: toNumber(pickValue(getFirstRecord(monthRes), [
        'totalCarbonEmission',
        'carbonEmission',
        'totalEmission',
        'emission'
      ])),
      day: toNumber(pickValue(getFirstRecord(dayRes), [
        'totalCarbonEmission',
        'carbonEmission',
        'totalEmission',
        'emission'
      ]))
    };

    const rankSource = Array.isArray(rankRes?.rows)
      ? rankRes.rows
      : Array.isArray(rankRes?.data)
        ? rankRes.data
        : Array.isArray(rankRes?.data?.rows)
          ? rankRes.data.rows
          : [];

    rankList.value = buildRankList(rankSource)
      .sort((a, b) => b.emissionValue - a.emissionValue)
      .slice(0, 8);
  } catch (error) {
    console.error('数据总览获取失败:', error);
  }
};

onMounted(() => {
  getOverviewData();
});
</script>

<style lang="scss" scoped>
.center_top_inner {
  display: flex;
  flex-direction: column;
  height: 100%;
  color: #e5f2ff;
}

.overview {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 12px;
}

.overview-row {
  display: flex;
  gap: 10px;
}

.overview-card {
  flex: 1;
  padding: 10px 12px;
  border-radius: 6px;
  background: rgba(18, 42, 84, 0.55);
  border: 1px solid rgba(64, 170, 255, 0.25);
  display: flex;
  flex-direction: column;
  justify-content: center;
  min-height: 62px;
}

.overview-title {
  font-size: 13px;
  color: #93bde6;
  margin-bottom: 6px;
}

.overview-value {
  font-size: 20px;
  font-weight: 600;
  color: #ffffff;
}

.rank-table {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: rgba(11, 24, 56, 0.35);
  border: 1px solid rgba(47, 126, 195, 0.35);
  border-radius: 6px;
  overflow: hidden;
}

.table-header,
.table-row {
  display: grid;
  grid-template-columns: 70px 1.4fr 1fr 1fr;
  align-items: center;
  padding: 8px 12px;
}

.table-header {
  background: rgba(25, 61, 138, 0.8);
  color: #cfe6ff;
  font-size: 13px;
}

.table-body {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.table-row {
  font-size: 12px;
  color: #e4f1ff;
}

.table-row:nth-child(2n) {
  background: rgba(255, 255, 255, 0.03);
}

.col-index {
  color: #7bd8ff;
}

.col-qty,
.col-emission {
  text-align: right;
}
</style>
