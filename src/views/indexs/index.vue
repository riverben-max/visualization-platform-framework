<template>
  <div class="contents" style="display: flex; justify-content: space-between; padding: 10px; width: 100%; height: 100vh; box-sizing: border-box; background: #0b1126;">
    
    <div class="contetn_left" style="width: 28%; display: flex; flex-direction: column; height: 100%;">
      
      <ItemWrap class="contetn_lr_item" title="能耗总览" style="height: 32%; margin-bottom: 15px;">
        <LeftTop />
      </ItemWrap>
      
      <ItemWrap class="contetn_lr_item" title="能耗占比" style="height: 32%; margin-bottom: 15px;">
        <LeftCenter />
      </ItemWrap>
      
      <ItemWrap class="contetn_lr_item" title="库存预警" style="height: 32%;">
        <LeftBottom/>
      </ItemWrap>
    </div>

    <div class="contetn_center" style="width: 42%; display: flex; flex-direction: column; height: 100%;">
      
      <ItemWrap class="contetn_lr_item" title="数据总览" style="height: 63%; margin-bottom: 15px;">
         </ItemWrap>
      
      <ItemWrap class="contetn_lr_item" title="销售计划" style="height: 35%;">
         </ItemWrap>
    </div>

    <div class="contetn_right" style="width: 28%; display: flex; flex-direction: column; height: 100%;">
      
      <ItemWrap class="contetn_lr_item" title="销售统计" style="height: 32%; margin-bottom: 15px;">
      </ItemWrap>
      
      <ItemWrap class="contetn_lr_item" title="销售排名（TOP8）" style="height: 32%; margin-bottom: 15px;">
      </ItemWrap>
      
      <ItemWrap class="contetn_lr_item" title="生产统计构成" style="height: 32%;">
      </ItemWrap>
    </div>

  </div>
</template>

<script>
import LeftCenter from "./left-center.vue";
import LeftTop from "./left-top.vue";
import LeftBottom from "./left-bottom.vue"

export default {
  components: {
    LeftCenter,
    LeftTop,
    LeftBottom
  },
  data() {
    return {

    };
  },
  filters: {
    numsFilter(msg) {
      return msg || 0;
    },
  },
  created() {
  },

  mounted() { },
  methods: {

  },
};
</script>
<style lang="scss" scoped>
// 内容
.contents {
  .contetn_left,
  .contetn_right {
    width: 540px;
    box-sizing: border-box;
    // padding: 16px 0;
  }

  .contetn_center {
    width: 720px;
  }

  //左右两侧 三个块
  .contetn_lr_item {
    height: 310px;
  }

  .contetn_center_top {
    width: 100%;
  }

  // 中间
  .contetn_center {
    display: flex;
    flex-direction: column;
    justify-content: space-around;
  }

  .contetn_center_bottom {
    height: 315px;
  }

  //左边 右边 结构一样
  .contetn_left,
  .contetn_right {
    display: flex;
    flex-direction: column;
    justify-content: space-around;
    position: relative;
  }
}

@keyframes rotating {
  0% {
    -webkit-transform: rotate(0) scale(1);
    transform: rotate(0) scale(1);
  }
  50% {
    -webkit-transform: rotate(180deg) scale(1.1);
    transform: rotate(180deg) scale(1.1);
  }
  100% {
    -webkit-transform: rotate(360deg) scale(1);
    transform: rotate(360deg) scale(1);
  }
}
</style>
