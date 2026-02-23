<template>
  <div class="right_bottom_inner">
    <dv-scroll-board class="scroll-board" :config="scrollConfig" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { currentGET } from '@/api/modules';

const labels = {
  index: '\u7f16\u53f7',
  product: '\u4ea7\u54c1',
  model: '\u578b\u53f7',
  spec: '\u89c4\u683c',
  unit: '\u5355\u4f4d',
  stock: '\u5e93\u5b58',
  produced: '\u5df2\u4ea7',
  pending: '\u5f85\u4ea7',
  emission: '\u78b3\u6392',
  placeholder: '--',
  fetchFail: '\u751f\u4ea7\u7edf\u8ba1\u6570\u636e\u83b7\u53d6\u5931\u8d25'
};

const scrollConfig = ref({
  header: [
    labels.index,
    labels.product,
    labels.model,
    labels.spec,
    labels.unit,
    labels.stock,
    labels.produced,
    labels.pending,
    labels.emission
  ],
  data: [],
  columnWidth: [70, 190, 170, 170, 120, 120, 120, 120, 130],
  align: ['center', 'left', 'left', 'left', 'center', 'right', 'right', 'right', 'right'],
  rowNum: 6,
  waitTime: 2200,
  carousel: 'single',
  headerBGC: 'rgba(25, 61, 138, 0.85)',
  oddRowBGC: 'rgba(255, 255, 255, 0.02)',
  evenRowBGC: 'rgba(255, 255, 255, 0.04)'
});

const normalizeList = (res) => {
  if (Array.isArray(res?.rows)) return res.rows;
  if (Array.isArray(res?.data)) return res.data;
  if (Array.isArray(res?.data?.rows)) return res.data.rows;
  return [];
};

const pickValue = (record, keys) => {
  for (const key of keys) {
    const value = record?.[key];
    if (value !== undefined && value !== null && value !== '') {
      return value;
    }
  }
  return undefined;
};

const formatNumber = (value) => {
  if (value === undefined || value === null || value === '') return '';
  const num = Number(value);
  return Number.isFinite(num) ? num.toLocaleString() : String(value);
};

const normalizeText = (value) => {
  if (value === undefined || value === null || value === '') return '';
  return String(value);
};

const joinPair = (left, right) => {
  const leftText = normalizeText(left);
  const rightText = normalizeText(right);
  if (!leftText && !rightText) return labels.placeholder;
  if (!leftText || rightText === labels.placeholder) return leftText || rightText;
  if (!rightText || leftText === labels.placeholder) return rightText;
  return `${leftText} / ${rightText}`;
};

const buildRows = (list) => {
  return list.map((item, index) => {
    const productCode = pickValue(item, [
      'productCode',
      'productNo',
      'materialCode',
      'productId',
      'code'
    ]);
    const productName = pickValue(item, [
      'productName',
      'materialName',
      'name'
    ]);
    const modelCode = pickValue(item, [
      'modelCode',
      'modelNo',
      'model',
      'typeCode',
      'type'
    ]);
    const modelName = pickValue(item, [
      'modelName',
      'typeName',
      'modelDesc'
    ]);
    const specCode = pickValue(item, [
      'specCode',
      'specNo',
      'specification',
      'spec'
    ]);
    const specName = pickValue(item, [
      'specName',
      'specContent',
      'specDesc'
    ]);
    const unitName = pickValue(item, [
      'unitName',
      'unit',
      'materialUnit'
    ]);
    const unitValue = pickValue(item, [
      'unitValue',
      'unitPrice',
      'unitAmount',
      'unitQty'
    ]);
    const stockName = pickValue(item, [
      'stockName',
      'warehouseName',
      'inventoryName'
    ]);
    const stockQty = pickValue(item, [
      'stock',
      'inventory',
      'totalInventory',
      'stockQty',
      'stockCount'
    ]);
    const producedName = pickValue(item, [
      'producedName',
      'completedName',
      'finishedName'
    ]);
    const producedQty = pickValue(item, [
      'produced',
      'producedQty',
      'completeNum',
      'finishedQty',
      'productionQty',
      'doneQty'
    ]);
    const pendingName = pickValue(item, [
      'pendingName',
      'todoName',
      'waitName'
    ]);
    const pendingQty = pickValue(item, [
      'pending',
      'pendingQty',
      'waitNum',
      'todoQty',
      'unProducedQty',
      'toProduce'
    ]);
    const emissionName = pickValue(item, [
      'emissionName',
      'carbonName'
    ]);
    const emissionQty = pickValue(item, [
      'carbonEmission',
      'totalCarbonEmission',
      'emission',
      'carbon',
      'carbonValue'
    ]);

    return [
      String(index + 1).padStart(2, '0'),
      joinPair(productCode, productName),
      joinPair(modelCode, modelName),
      joinPair(specCode, specName),
      joinPair(unitName, formatNumber(unitValue)),
      joinPair(stockName, formatNumber(stockQty)),
      joinPair(producedName, formatNumber(producedQty)),
      joinPair(pendingName, formatNumber(pendingQty)),
      joinPair(emissionName, formatNumber(emissionQty))
    ];
  });
};

const buildPlaceholderRows = (count) => {
  const row = [
    labels.placeholder,
    labels.placeholder,
    labels.placeholder,
    labels.placeholder,
    labels.placeholder,
    labels.placeholder,
    labels.placeholder,
    labels.placeholder
  ];
  return Array.from({ length: count }, (_, index) => [
    String(index + 1).padStart(2, '0'),
    ...row
  ]);
};

const getData = async () => {
  try {
    const res = await currentGET('big5');
    //console.log(res);
    
    if (!res?.success) {
      scrollConfig.value = {
        ...scrollConfig.value,
        data: buildPlaceholderRows(8)
      };
      return;
    }

    const list = normalizeList(res);
    const rows = buildRows(list);
    scrollConfig.value = {
      ...scrollConfig.value,
      data: rows.length ? rows : buildPlaceholderRows(8)
    };
  } catch (error) {
    console.error(labels.fetchFail, error);
    scrollConfig.value = {
      ...scrollConfig.value,
      data: buildPlaceholderRows(8)
    };
  }
};

onMounted(() => {
  getData();
});
</script>

<style lang="scss" scoped>
.right_bottom_inner {
  width: 100%;
  height: 100%;
}

.scroll-board {
  width: 100%;
  height: 100%;
}
</style>
