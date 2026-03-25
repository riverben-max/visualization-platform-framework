<template>
  <div class="overview">
    <div class="item">
      <div class="box power">
        <dv-digital-flop :config="powerConfig" style="width:100%;height:100%" />
        <p>耗电量</p>
      </div>
    </div>
    <div class="item">
      <div class="box water">
        <dv-digital-flop :config="waterConfig" style="width:100%;height:100%" />
      </div>
      <p>耗水量</p>
    </div>
    <div class="item">
      <div class="box carbon">
        <dv-digital-flop :config="carbonConfig" style="width:100%;height:100%" />
      </div>
      <p>碳排放量</p>
    </div>
  </div>
</template>
<script setup>
import { onMounted, ref } from 'vue';
import { currentGET } from '../../api/modules';

const powerConfig = ref({
  number: [100],
  content: '{nt}',
  style: {
    fontSize: '17'
  }
})

const waterConfig = ref({
  number: [100],
  content: '{nt}',
  style: {
    fontSize: '17'
  }
})

const carbonConfig = ref({
  number: [100],
  content: '{nt}',
  style: {
    fontSize: '17'
  }
})

const getData = async () => {
  const res = await currentGET('big2')

  if (!res || !res.rows[0]) return
  const rows = res.rows[0]//!!!!小心rows
  const power = rows.totalOfficePowerConsume + rows.totalPowerConsume
  const water = rows.totalOfficeWaterConsume + rows.totalWaterConsume
  const carbon = rows.totalCarbonEmission

  powerConfig.value.number = [power]
  waterConfig.value.number = [water]
  carbonConfig.value.number = [carbon]
}

onMounted(() => {
  getData()
})
</script>
<style scoped>
.overview {
  display: flex;
  justify-content: space-around;
}

p {
  text-align: center;
}

.box {
  width: 160px;
  height: 200px;
  position: relative;
  z-index: 1;
}

.box::before {
  content: '';
  width: 100%;
  height: 100%;
  position: absolute;
  z-index: -1;
  background-repeat: no-repeat;
  background-size: contain;
  background-position: center;
}

.power::before {
  background-image: url('@/assets/img/left_top_lv.png');
}

.water::before {
  background-image: url('@/assets/img/left_top_huang.png');
}

.carbon::before {
  background-image: url('@/assets/img/left_top_lan.png');
}
</style>