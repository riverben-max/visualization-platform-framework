<!-- <template>
  <div class="right_center_inner">
    <Echart v-if="pageflag" id="rightCenter" :options="options" />
    <Reacquire v-else @onclick="getData" style="line-height: 200px">
      {{ labels.retry }}
    </Reacquire>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { currentGET } from '@/api/modules';
import Echart from '@/components/echart/index.vue';
import Reacquire from '@/components/reacquire/reacquire.vue';

const labels = {
  retry: '\u91cd\u65b0\u83b7\u53d6',
  salesAmountAxis: '\u9500\u552e\u989d',
  salesAmountSeries: '\u9500\u552e\u989d',
  unknownCompany: '--',
  fetchFail: '\u9500\u552e\u6392\u540d\u6570\u636e\u83b7\u53d6\u5931\u8d25'
};

const pageflag = ref(true);
const options = ref({});
const salesList = ref([]);

const toNumber = (value) => {
  const num = Number(value);
  return Number.isFinite(num) ? num : 0;
};

const formatNumber = (value) => {
  const num = Number(value);
  if (!Number.isFinite(num)) return '--';
  return num.toLocaleString();
};

const pickValue = (record, keys) => {
  for (const key of keys) {
    const val = record?.[key];
    if (val !== undefined && val !== null && val !== '') {
      return val;
    }
  }
  return undefined;
};

// const normalizeList = (res) => {
//   if (Array.isArray(res?.rows)) return res.rows;
//   if (Array.isArray(res?.data)) return res.data;
//   if (Array.isArray(res?.data?.rows)) return res.data.rows;
//   return [];
// };

const buildSalesList = (list) => {
  return list.map((item, index) => {
    const name = pickValue(item, [
      'customerName',
      'companyName',
      'saleCustomerName',
      'customer',
      'company',
      'name'
    ]);
    const amount = pickValue(item, [
      'saleAmount',
      'salesAmount',
      'totalSales',
      'saleMoney',
      'amount',
      'value',
      'sales',
      'sale'
    ]);
    return {
      key: item?.id || `${index + 1}`,
      name: name || labels.unknownCompany,
      value: toNumber(amount)
    };
  });
};

const initChart = () => {
  const colors = [
    '#3ba1ff',
    '#4fd97f',
    '#f6bd16',
    '#f58b3c',
    '#8b7bff',
    '#2ec7c9',
    '#ff6b6b',
    '#7bd8ff'
  ];
  const names = salesList.value.map((item) => item.name);
  const values = salesList.value.map((item) => item.value);

  options.value = {
    grid: {
      left: 120,
      right: 40,
      top: 20,
      bottom: 20,
      containLabel: true
    },
    
    xAxis: {
      type: 'value',
      name: labels.salesAmountAxis,
      axisLine: { lineStyle: { color: 'rgba(255,255,255,0.3)' } },
      axisLabel: { color: '#cfe6ff' },
      splitLine: { lineStyle: { color: 'rgba(255,255,255,0.08)' } }
    },
    yAxis: {
      type: 'category',
      data: names,
      inverse: true,
      axisTick: { show: false },
      axisLine: { lineStyle: { color: 'rgba(255,255,255,0.3)' } },
      axisLabel: { color: '#cfe6ff', fontSize: 12 }
    },
    series: [
      {
        name: labels.salesAmountSeries,
        type: 'bar',
        barWidth: 14,
        data: values,
        itemStyle: {
          borderRadius: [0, 6, 6, 0],
          color: (params) => colors[params.dataIndex % colors.length]
        },
        label: {
          show: true,
          position: 'right',
          color: '#ffffff',
          fontSize: 12,
          formatter: (params) => formatNumber(params.value)
        }
      }
    ]
  };
};

const getData = async () => {
  pageflag.value = true;
  try {
    const res = await currentGET('big7');
    console.log(res);
    
    if (!res?.success) {
      pageflag.value = false;
      return;
    }

    const list = buildSalesList(normalizeList(res))
      .sort((a, b) => b.value - a.value)
      .slice(0, 8);

    if (!list.length) {
      pageflag.value = false;
      return;
    }

    salesList.value = list;
    initChart();
  } catch (error) {
    console.error(labels.fetchFail, error);
    pageflag.value = false;
  }
};

onMounted(() => {
  getData();
});
</script>

<style lang="scss" scoped>
.right_center_inner {
  width: 100%;
  height: 100%;
}
</style> -->












<!-- 柱状图最小模板 -->
<template>
  <div class="right-center-inner">
    <Echart v-if="pageFlag" :options="options" />

  </div>
</template>
<script setup>
import { onMounted, ref } from 'vue';

const pageFlag = ref(true)
const options = ref({})
const chartData = ref({
  names: ['zhangsan', 'wangwu', 'lisi'],
  values: ['100', '233', '312']
})
const getData = async () => {

}
const initChart = () => {
  const colors = [
    '#3ba1ff',
    '#4fd97f',
    '#f6bd16',
    '#f58b3c',
    '#8b7bff',
    '#2ec7c9',
    '#ff6b6b',
    '#7bd8ff'
  ];
  // const names = salesList.value.map((item) => item.name);
  // const values = salesList.value.map((item) => item.value);
  options.value = {
    grid: {
      left: 20,
      right: 60,
      top: 40,
      bottom: 20,
      containLabel: true
    },
    xAxis: {
      name: '销售额',
      type: 'value',
      // splitLine: { lineStyle: { color: 'rgba(255,255,255,0.08)' } },
      // axisLine: { lineStyle: { color: 'rgba(255,255,255,0.3)' } },
      data: chartData.value.values
    },
    yAxis: {
      name: '销售公司名称',
      type: 'category',
      data: chartData.value.names,
    },
    series: {
      name: '销售排名',
      type: 'bar',
      barWidth: 14,
      data: chartData.value.values
    }
  }

}
onMounted(() => {
  initChart()
})
</script>
<style lang="scss" scoped>
.right-center-inner {
  background-color: aquamarine;
  height: 250px;
}
</style>