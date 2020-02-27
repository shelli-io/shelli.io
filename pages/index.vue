<template>
  <div class="container">
    <div class="section hero">
      <img class="logo" src="@/assets/shelli-logo.svg" />

      <h1 class="title">shelli</h1>
      <h2 class="subtitle">Accounting as a Service</h2>
    </div>
    <div class="section demo">
      <h2 class="subtitle">IOTA Invoice Demonstrator</h2>
      <p>
        Creates an invoice PDF and adds Proof of Existence within the Tangle.
      </p>
      <br />
      <InvoiceForm />
    </div>
  </div>
</template>

<script>
import InvoiceForm from '@/components/InvoiceForm'

export default {
  components: { InvoiceForm }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Heebo|Open+Sans&display=swap');

.container {
  font-family: 'Open Sans', sans-serif;
  margin: 0 auto;
  min-height: 100vh;
  justify-content: center;
  align-items: center;
  text-align: center;
  width: 100%;
}

.section {
  height: 100vh;
  width: 100%;
}

.hero {
  margin-top: 30%;
}

.title {
  font-family: 'Heebo', sans-serif;
  display: block;
  font-weight: 500;
  font-size: 100px;
  color: #35495e;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #35495e;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.logo {
  height: 15vh;
}
</style>
