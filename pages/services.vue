<template>
  <div class="services">
    <h1 class="title">Services</h1>
    <el-tabs v-model="activeName" @tab-click="handleClick" class="tabs">
      <el-tab-pane class="tab-pane" label="Invoice Generator" name="first">
        <h2>IOTA Invoice Demonstrator</h2>
        <p class="description">
          Creates an invoice PDF and adds Proof of Existence within the IOTA
          Tangle.
        </p>

        <InvoiceGenerator />
      </el-tab-pane>
      <el-tab-pane class="tab-pane" label="Invoice Verifier" name="second">
        <h2>IOTA Invoice Verifier</h2>
        <p class="description">
          Verifies an invoice PDF Proof of Existence within the IOTA Tangle.
        </p>
        <InvoiceVerifier />
      </el-tab-pane>
      <el-tab-pane class="tab-pane" label="Price Ticker" name="third">
        <h2>IOTA Price Ticker</h2>
        <p class="description">
          Loads data from the shelli.io API - which gets the data from different
          exchanges and publishes the data in a MAM Stream.
        </p>
        <CryptoPriceTracker />
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import InvoiceGenerator from '@/components/InvoiceGenerator'
import InvoiceVerifier from '@/components/InvoiceVerifier'
import CryptoPriceTracker from '@/components/CryptoPriceTracker'

export default {
  components: { InvoiceGenerator, InvoiceVerifier, CryptoPriceTracker },
  data() {
    return {
      activeName: 'first'
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
