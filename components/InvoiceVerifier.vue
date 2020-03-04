<template>
  <div>
    <h3>Please add your invoice data to be verified</h3>

    <el-form :model="form" label-position="top" label-width="100px">
      <el-form-item class="label" label="Select your invoice PDF">
        <input @change="processFile($event)" type="file" />
      </el-form-item>
      <el-form-item class="label" label-position="top" label="IOTA Address">
        <el-input v-model="form.address"></el-input>
      </el-form-item>
      <el-form-item class="label" label-position="top" label="Transaction Hash">
        <el-input v-model="form.tx_hash"></el-input>
      </el-form-item>
      <el-button
        @click="checkInvoice"
        v-loading.fullscreen.lock="loading"
        type="primary"
        >Validate invoice</el-button
      >
    </el-form>
    <div v-if="result">
      <h2>Your Result</h2>
      <div v-if="verified">
        <el-alert title="The invoice got verified" type="success"></el-alert>
      </div>
      <div v-else>
        <el-alert
          title="Could not verifie your invoice. There is something wrong."
          type="error"
        ></el-alert>
      </div>
    </div>
  </div>
</template>

<script>
const { composeAPI } = require('@iota/core')
const { trytesToAscii } = require('@iota/converter')
const sha256 = require('js-sha256')

export default {
  components: {},
  data() {
    return {
      form: {
        address: '',
        tx_hash: ''
      },
      file: '',
      loading: false,
      verified: null,
      result: false
    }
  },
  methods: {
    processFile(event) {
      this.file = event.target.files[0]
    },
    async checkInvoice() {
      this.loading = true
      this.result = false
      console.log('form', this.form)
      const bundle = {
        provider: 'https://nodes.devnet.thetangle.org:443',
        address: this.form.address,
        hash: this.form.tx_hash
      }
      this.verified = await this.verify(bundle, this.file)
      console.log('verified', this.verified)
      this.loading = false
      this.result = true
    },

    fileAdded(file) {
      console.log('file', file)
      this.file = file
    },
    async fetchTx(bundle) {
      const iota = composeAPI({
        provider: bundle.provider
      })
      console.log('iota', iota)
      const address = bundle.address
      let response = null
      try {
        response = await iota.findTransactionObjects({ addresses: [address] })
      } catch (err) {
        throw err
      }

      /*
    We need both address and transaction Hash(the latter to use it to filter the exact tx)
  */
      if (
        response &&
        response !== null &&
        response !== '' &&
        response !== undefined
      ) {
        const asciiArr = response.filter((res) => res.hash === bundle.hash)[0]
        if (!asciiArr || asciiArr === undefined) {
          console.log('Returned an empty object')
          return
        }
        return trytesToAscii(`${asciiArr.signatureMessageFragment}9`)
      } else {
        return null
      }
    },
    async verify(bundle, file) {
      // const calculatedHash = hash(docpath, isBinaryInput)
      console.log('file.arrayBuffer()', file.arrayBuffer())
      const buffer = await file.arrayBuffer()
      console.log('buffer', buffer)
      const calculatedHash = sha256(buffer)
      console.log('calculatedHash', calculatedHash)
      let tangleHash = await this.fetchTx(bundle)
      console.log('tangleHash', tangleHash)
      if (!tangleHash) return 0
      tangleHash = tangleHash.replace(/\0/g, '')
      // tangleHash.replace(/\0/g, '') removes u0000
      return calculatedHash.trim() === tangleHash.trim()
    }
  }
}
</script>

<style lang="scss">
h3 {
  margin-bottom: 30px;
}
.el-form-item__label {
  float: left !important;
  width: 100%;
  padding: 0 !important;
}
</style>
