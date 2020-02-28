<template>
  <div class="services">
    <h1>shelli demo services</h1>
    <el-tabs v-model="activeName" @tab-click="handleClick">
      <el-tab-pane label="Invoice Generator" name="first">
        <h2>Invoice Generator</h2>
        <h2 class="subtitle">IOTA Invoice Demonstrator</h2>
        <p>
          Creates an invoice PDF and adds Proof of Existence within the Tangle.
        </p>
        <InvoiceForm />
      </el-tab-pane>
      <el-tab-pane label="Invoice Verifier" name="second">
        <h2>Invoice Verifier</h2>
        <InvoiceVerifier />
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import InvoiceForm from '@/components/InvoiceForm'
import InvoiceVerifier from '@/components/InvoiceVerifier'

export default {
  components: { InvoiceForm, InvoiceVerifier },
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
