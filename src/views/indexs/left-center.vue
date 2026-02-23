<template>
  <div class="left_center_inner">
    <div v-if="pageflag" id="leftCenter" ref="chartRef"></div>
    <Reacquire v-else @click="getData" style="line-height: 200px">
      重新获取
    </Reacquire>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onBeforeUnmount, nextTick } from 'vue';
import * as echarts from 'echarts';
import { currentGET } from '@/api/modules';

// --- 响应式数据 ---
const pageflag = ref(true);
const options = ref({});
const chartRef = ref(null);
const chartInstance = ref(null);
const timer = ref(null);

const consumeData = reactive({
  totalPowerConsume: 0,
  totalOfficePowerConsume: 0,
  totalWaterConsume: 0,
  totalOfficeWaterConsume: 0
});

// --- 核心逻辑：初始化图表配置 (根据截图补全) ---
const initChart = () => {

  // 1. 计算总数

  const total = consumeData.totalPowerConsume +
    consumeData.totalOfficePowerConsume +
    consumeData.totalWaterConsume +
    consumeData.totalOfficeWaterConsume;
  const colors = ["#ECA444", "#33A1DB", "#56B557", "#c66863"];
  //console.log(consumeData);
  



  // 2. 补全 options 配置
  options.value = {
    series: [
      {
        name: '能耗占比',
        type: 'pie',
        radius: ['35%', '65%'],
        avoidLabelOverlap: true,
        itemStyle: {
          borderRadius: 5,
          borderColor: 'rgba(0,0,0,0)',
          borderWidth: 1
        },
        colors: colors,
        data: [
          { value: consumeData.totalPowerConsume, name: '生产耗电量' },
          { value: consumeData.totalOfficeWaterConsume, name: '办公用水量' },
          { value: consumeData.totalWaterConsume, name: '生产用水量' },
          { value: consumeData.totalOfficePowerConsume, name: '办公耗电量' },
        ],
        label: {
          show: true,
          /* 
          第二部分 {c|{c}}：{c} 是 数值（Value）。这里填入 “35200”。外面的 {c|...} 是为了给它穿上名为 c 的样式（比如蓝色字）。
          第三部分 {per|{d}%}：{d} 是 百分比（Percent）。这里填入 “17.15”。最后的 % 就是个普通的百分号。外面的 {per|...} 同样是套样式。 */
          formatter: "{b|{b}} \n {c|{c}} {per|{d}%}",
          rich: {
            b: {
              color: '#fff',
              fontSize: 16,
              lineHeight: 26
            },
            c: {
              color: 'pink',
              fontSize: 14,
            },
            per: {
              color: 'red',
              fontSize: 14,
            },
          }
        },
        labelLine: {
          length: 20,
          length2: 36,
          show: true
        },
      }
    ],

    title: {
      // 截图中的特殊格式：数值 | 总数
      text: [`{value|${total}}`, "{name|总数}"].join("\n"),
      top: "center",
      left: "center",
      textStyle: {
        rich: {
          value: {
            color: "#fff",
            fontSize: 24,
            fontWeight: "bold",
            lineHeight: 20
          },
          name: {
            color: "#fff",
            lineHeight: 20
          }
        }
      }
    },
    tooltip: {
      trigger: 'item',
      backgroundColor: "rgba(0,0,0,.6)",
      borderColor: "rgba(147,235,248,.8)",
      textStyle: {
        color: "#fff"
      }
    },
    legend: {
      show: true,
      top: "5%",      // 距离顶部 5%
      left: "5%",     // 【修改点】距离左侧 5% (留出一点空隙，不要贴边)
      orient: 'vertical',
      textStyle: {
        color: "#fff",
        fontSize: 12
      },
    },
  }
};

const renderChart = () => {
  if (!chartRef.value) return;
  if (chartInstance.value && chartInstance.value.getDom() !== chartRef.value) {
    chartInstance.value.dispose();
    chartInstance.value = null;
  }
  if (!chartInstance.value) {
    chartInstance.value = echarts.init(chartRef.value);
  }
  chartInstance.value.setOption(options.value, true);
};
/* options.value = {
    title: {
      // 截图中的特殊格式：数值 | 总数
      text: [`{value|${total}}`, "{name|总数}"].join("\n"),
      top: "center",
      left: "center",
      textStyle: {
        rich: {
          value: {
            color: "#fff",
            fontSize: 24,
            fontWeight: "bold",
            lineHeight: 20
          },
          name: {
            color: "#fff",
            lineHeight: 20
          }
        }
      }
    },
    tooltip: {
      trigger: 'item',
      backgroundColor: "rgba(0,0,0,.6)",
      borderColor: "rgba(147,235,248,.8)",
      textStyle: {
        color: "#fff"
      }
    },
    legend: {
      show: false,
      top: "5%",
      left: "center"
    },
    series: [
      {
        name: '能耗占比',
        type: 'pie',
        radius: ['42%', '65%'],
        avoidLabelOverlap: true,
        itemStyle: {
          borderRadius: 4,
          borderColor: "rgba(0,0,0,0)",
          borderWidth: 2
        },
        color: colors,
        label: {
          show: true,
          // 截图中的富文本格式
          formatter: "{b|{b}} \n {c|{c}} {per|{d}%}",
          rich: {
            b: {
              color: "#fff",
              fontSize: 12,
              lineHeight: 26
            },
            c: {
              color: "#31ABE3",
              fontSize: 14
            },
            per: {
              color: "#31ABE3",
              fontSize: 14
            }
          }
        },
        labelLine: {
          length: 20,
          length2: 36,
          show: true
        },
        data: [
          { value: consumeData.totalWaterConsume, name: '生产用水量' },
          { value: consumeData.totalPowerConsume, name: '生产耗电量' },
          { value: consumeData.totalOfficePowerConsume, name: '办公耗电量' },
          { value: consumeData.totalOfficeWaterConsume, name: '办公用水量' }
        ]
      }
    ]
  }; */
// --- 业务逻辑 ---

const getData = async () => {
  pageflag.value = true;
  try {
    const res = await currentGET('big1');
    if (res.success) {
      // 更新数据
      Object.assign(consumeData, res.data[0]);


      // 数据更新后重新渲染图表
      initChart();

      nextTick(() => {
        renderChart();
        switper();
      });
    } else {
      pageflag.value = false;
    }
  } catch (error) {
    console.error("数据获取失败:", error);
    pageflag.value = false;
  }
};

const switper = () => {
  if (timer.value) return;
  const looper = () => { getData(); };
  const autoTime = 3000;
  timer.value = setInterval(looper, autoTime);

  const myChart = chartInstance.value;
  if (myChart) {
    myChart.off('mouseover');
    myChart.off('mouseout');
    myChart.on('mouseover', () => {
      clearInterval(timer.value);
      timer.value = null;
    });
    myChart.on('mouseout', () => {
      if (!timer.value) timer.value = setInterval(looper, autoTime);
    });
  }
};

onMounted(() => {
  getData();
});

onBeforeUnmount(() => {
  if (timer.value) clearInterval(timer.value);
  if (chartInstance.value) {
    chartInstance.value.dispose();
    chartInstance.value = null;
  }
});
</script>

<style lang="scss" scoped>
.left_center_inner {
  width: 100%;
  height: 100%;
}

#leftCenter {
  width: 100%;
  height: 100%;
}
</style>
