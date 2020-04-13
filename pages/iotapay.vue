<template>
  <div class="services">
    <h1 class="title">IOTAPAY</h1>
    <IotaPay />
  </div>
</template>

<script>
import IotaPay from '@/components/IotaPay'

export default {
  components: { IotaPay },
  data() {
    return {
      address: ''
    }
  },
  methods: {
    handleClick(tab, event) {
      console.log(tab, event)
    }
  }
}
</script>

<style lang="scss">
.services {
  max-width: 800px;
  padding: 0 20px;
  .title {
    font-size: 4em;
    margin-top: 80px;
  }
  .tabs {
    margin-top: 40px;
    .tab-pane {
      margin-top: 20px;
    }
  }
  .el-tabs__item {
    font-size: 1.5em;
  }
  .description {
    margin-bottom: 40px;
  }
}
</style>
