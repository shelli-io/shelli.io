<template>
  <div>
    <h3>IOTA Price: ${{ price }}</h3>
    <p>Snapshot from: {{ time }}</p>
    <p>
      MAM Root:
      <a
        :href="
          `https://mam-explorer.firebaseapp.com/?provider=https://nodes.devnet.iota.org&mode=public&root=${root}`
        "
        target="_blank"
        >MAM Explorer
      </a>
    </p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      time: '',
      price: '',
      root: ''
    }
  },
  mounted() {
    this.$axios
      .get(process.env.VUE_APP_SHELLI_API_URL + '/getLatestSnapshot')
      .then((resp) => {
        console.log('resp', resp)
        if (resp.status === 200) {
          const newDate = new Date()
          newDate.setTime(resp.data.timestamp)
          this.time = newDate.toUTCString()
          this.price = resp.data.data[0]
          this.root = resp.data.root
        }
      })
      .catch((err) => {
        console.log('err', err)
      })
  }
}
</script>

<style></style>
