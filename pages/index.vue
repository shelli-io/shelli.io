<template>
  <div class="container">
    <div>
      <img class="logo" src="@/assets/shelli-logo.svg" />

      <h1 class="title">shelli</h1>
      <h2 class="subtitle">economy as a service</h2>
    </div>
  </div>
</template>

<script>
export default {
  components: {}
}
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Heebo|Open+Sans&display=swap');

.container {
  font-family: 'Open Sans', sans-serif;
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Heebo', sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.logo {
  height: 15vh;
}
</style>
