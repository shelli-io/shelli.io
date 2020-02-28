<template>
  <div class="services">
    <h1>shelli demo services</h1>
    <el-tabs v-model="activeName" @tab-click="handleClick">
      <el-tab-pane label="Invoice Generator" name="first">
        <h2>IOTA Invoice Demonstrator</h2>
        <p>
          Creates an invoice PDF and adds Proof of Existence within the Tangle.
        </p>
        <InvoiceGenerator />
      </el-tab-pane>
      <el-tab-pane label="Invoice Verifier" name="second">
        <h2>Invoice Verifier</h2>
        <InvoiceVerifier />
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import InvoiceGenerator from '@/components/InvoiceGenerator'
import InvoiceVerifier from '@/components/InvoiceVerifier'

export default {
  components: { InvoiceGenerator, InvoiceVerifier },
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

<style></style>
