<template>
  <div>
    <p>Buying IOTA never was that easy.</p>
    <div v-if="balance == 0">
      <h3>IOTA balance: {{ balance }}i</h3>
      <p>Paypal env {{ paypalEnv }}</p>
      <no-ssr>
        <paypal-checkout
          :env="paypalEnv"
          :client="credentials"
          amount="10.00"
          currency="EUR"
        ></paypal-checkout>
      </no-ssr>
    </div>
    <div v-else>
      <p>Service currently disabled. Try again later.</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      balance: null,
      paypalEnv: process.env.paypalEnv,
      credentials: {
        sandbox: process.env.paypalSandboxId,
        production: process.env.paypalProductionId
      }
    }
  },
  mounted() {
    this.$axios
      .get(process.env.iotapayUrl + '/balance')
      .then((resp) => {
        console.log('resp', resp)
        if (resp.status === 200) {
          console.log('success')
          this.balance = resp.data.balance
        }
      })
      .catch((err) => {
        console.log('err', err)
      })
  }
}
</script>

<style></style>
