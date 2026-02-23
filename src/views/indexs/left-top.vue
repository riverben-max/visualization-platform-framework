<template>
  <ul class="user_overview flex">
    <li style="color: #00fdfa;">
      <div class="user_overview_nums allnum">
        <dv-digital-flop :config="powerConsumeCfg" style="width:100%;height:100%;" />
      </div>
      <p>耗电量</p>
    </li>
    <li style="color: #07f7a8;">
      <div class="user_overview_nums online">
        <dv-digital-flop :config="waterConsumeCfg" style="width:100%;height:100%;" />
      </div>
      <p>耗水量</p>
    </li>
    <li style="color: #e3b337;">
      <div class="user_overview_nums offline">
        <dv-digital-flop :config="carbonEmissionCfg" style="width:100%;height:100%;" />
      </div>
      <p>碳排放量</p>
    </li>
  </ul>
</template>
<script>
import { currentGET } from '../../api/modules';

export default {
  data() {
    return {
      // 对应图1的数据结构配置
      powerConsumeCfg: {
        number: [100], // 初始值
        content: '{nt}',
        
        style: {
          fontSize: 24,
          fill: '#00fdfa'
        }
      },
      waterConsumeCfg: {
        number: [100],
        content: '{nt}',
        style: {
          fontSize: 24,
          fill: '#07f7a8'
        }
      },
      carbonEmissionCfg: {
        number: [100],
        content: '{nt}',
        style: {
          fontSize: 24,
          fill: '#e3b337'
        }
      }
    };
  },

  created() {
    this.getData();
    // 先设置测试数据，验证组件是否能正常显示
    //console.log('组件创建，初始配置：', this.powerConsumeCfg);
  },
  methods: {
    getData() {
      //console.log('开始获取数据...');
      currentGET('big2').then(res => {
        // 在 getData 方法中
        if (res.code == 200) {
          let emissionOverview = res.rows[0];

          // 1. 强制转为 Number，防止字符串拼接（例如 "100" + "20" 变成 "10020"）
          let wateConsume = Number(emissionOverview.totalWaterConsume || 0) + Number(emissionOverview.totalOfficeWaterConsume || 0);
          let powerConsume = Number(emissionOverview.totalOfficePowerConsume || 0) + Number(emissionOverview.totalPowerConsume || 0);

          // 检查一下取值逻辑
          let carbonEmission = Number(emissionOverview.totalCarbonEmission || 0);

          //console.log('数值检查:', { powerConsume, wateConsume, carbonEmission });

          // 2. 更新配置
          this.powerConsumeCfg = {
            ...this.powerConsumeCfg,
            number: [powerConsume]
          };

          this.waterConsumeCfg = {
            ...this.waterConsumeCfg,
            number: [wateConsume]
          };

          this.carbonEmissionCfg = {
            ...this.carbonEmissionCfg,
            number: [carbonEmission]
          };
        }
      })
    }
  }
};
</script>

<style lang='scss' scoped>
.user_overview {
  display: flex;
  /* 确保 ul 是 flex 布局 */
  justify-content: space-around;
  padding: 0;

  li {
    list-style: none;
    /* 去掉圆点 */
    flex: 1;

    p {
      text-align: center;
      height: 16px;
      font-size: 16px;
      margin-top: 10px;
      /* 给文字一点间距 */
    }

    .user_overview_nums {
      width: 160px;
      height: 160px;
      margin: 40px auto 10px;
      /* 调整间距 */

      /* --- 关键修改 1: 去掉 line-height --- */
      /* line-height: 100px;  <-- 删掉这一行，这会干扰组件渲染 */

      position: relative;
      z-index: 1;
      /* 建立层级上下文 */

      /* 让内部组件居中 */
      display: flex;
      justify-content: center;
      align-items: center;

      /* --- 关键修改 2: 伪元素层级下沉 --- */
      &::before {
        content: "";
        position: absolute;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;
        z-index: -1;
        /* <--- 必须加这个！否则背景图会挡住数字 */
        background-size: contain;
        /* 建议改为 contain 防止图片变形 */
        background-repeat: no-repeat;
        background-position: center;
      }
    }

    /* 背景图部分保持不变 */
    .allnum::before {
      background-image: url("../../assets/img/left_top_lv.png");
    }

    .online::before {
      background-image: url("../../assets/img/left_top_huang.png");
    }

    .offline::before {
      background-image: url("../../assets/img/left_top_lan.png");
    }
  }
}
</style>
