<template>
  <div>
    <h3>Please add your invoice data to be verified</h3>

    <el-form :model="form" label-position="top" label-width="100px">
      <el-form-item label="Select your file">
        <dropzone
          id="foo"
          ref="el"
          :options="options"
          :destroyDropzone="true"
          v-on:vdropzone-file-added="fileAdded"
        ></dropzone>
      </el-form-item>
      <el-form-item label="IOTA Address">
        <el-input v-model="form.address"></el-input>
      </el-form-item>
      <el-form-item label="Transaction Hash">
        <el-input v-model="form.tx_hash"></el-input>
      </el-form-item>
      <el-button @click="checkInvoice" type="primary"
        >Validate invoice</el-button
      >
    </el-form>
  </div>
</template>

<script>
import Dropzone from 'nuxt-dropzone'
import 'nuxt-dropzone/dropzone.css'
const { composeAPI } = require('@iota/core')
const { trytesToAscii } = require('@iota/converter')
const sha256 = require('js-sha256')

export default {
  components: {
    Dropzone
  },
  data() {
    return {
      // See https://rowanwins.github.io/vue-dropzone/docs/dist/index.html#/props
      options: {
        url: 'http://httpbin.org/anything'
      },
      form: {
        address: '',
        tx_hash: ''
      },
      file: ''
    }
  },
  mounted() {
    // Everything is mounted and you can access the dropzone instance
    const instance = this.$refs.el.dropzone
    console.log('instance', instance)
  },
  methods: {
    async checkInvoice() {
      console.log('form', this.form)
      const bundle = {
        provider: 'https://altnodes.devnet.iota.org',
        address: this.form.address,
        hash: this.form.tx_hash
      }
      const verified = await this.verify(bundle, this.file)
      console.log('verified', verified)
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
      tangleHash = tangleHash.replace(/\0/g, '')
      // tangleHash.replace(/\0/g, '') removes u0000
      return calculatedHash.trim() === tangleHash.trim()
    }
  }
}
</script>

<style></style>
