<template>
  <el-form
    ref="form"
    :model="form"
    :rules="rules"
    status-icon
    label-width="120px"
    class="form"
  >
    <div id="invoice" class="invoice-box">
      <table cellpadding="0" cellspacing="0">
        <tr class="top">
          <td colspan="4">
            <table>
              <tr>
                <td class="title">
                  <img
                    src="@/assets/shelli-logo.svg"
                    style="width:100%; max-width:80px;"
                  />
                </td>

                <td contenteditable="true">
                  Invoice #: 123<br />
                  Created: February 29, 2020<br />
                  Due: February 29, 2020
                </td>
              </tr>
            </table>
          </td>
        </tr>

        <tr class="information">
          <td colspan="4">
            <table>
              <tr>
                <td contenteditable="true">
                  shelli, Inc.<br />
                  39527 Sunny Road<br />
                  Sunnyville, CA 39527
                </td>

                <td contenteditable="true">
                  JD Corp.<br />
                  John Doe<br />
                  john@example.com
                </td>
              </tr>
            </table>
          </td>
        </tr>

        <tr class="heading">
          <td colspan="3">Payment Method</td>
          <td>MIOTA Price</td>
        </tr>

        <tr class="details">
          <td colspan="3">IOTA</td>
          <td>{{ current_iota_price }} €</td>
        </tr>

        <tr class="heading">
          <td>Item</td>
          <td>Unit Cost</td>
          <td>Quantity</td>
          <td>Price</td>
        </tr>

        <tr v-for="(item, index) in items" :key="index" class="item">
          <td><input v-model="item.description" /></td>
          <td><input v-model="item.price" type="number" /> €</td>
          <td><input v-model="item.quantity" type="number" /></td>
          <td>{{ (item.price * item.quantity) | currency }} €</td>
        </tr>

        <tr>
          <td colspan="4">
            <div @click="addRow" class="btn-add-row">Add row</div>
          </td>
        </tr>

        <tr class="total">
          <td colspan="3"></td>
          <td>Total: {{ total | currency }} €</td>
        </tr>
      </table>
    </div>

    <el-form-item>
      <el-button @click="submitForm('form')" type="primary"
        >Create invoice</el-button
      >
      <el-button @click="resetForm('form')">Reset</el-button>
    </el-form-item>
    <pre>Loading: {{ loading }}</pre>
  </el-form>
</template>
<script>
const { composeAPI } = require('@iota/core')
const { asciiToTrytes } = require('@iota/converter')
const sha256 = require('js-sha256')
const axios = require('axios')

export default {
  filters: {
    currency(value) {
      return value.toFixed(2)
    }
  },
  data() {
    return {
      loading: false,
      form: {},
      rules: {},
      current_iota_price: null,
      items: [
        { description: 'Website design', quantity: 1, price: 300 },
        { description: 'Website design', quantity: 1, price: 75 },
        { description: 'Website design', quantity: 1, price: 10 }
      ]
    }
  },
  computed: {
    total() {
      return this.items.reduce(
        (acc, item) => acc + item.price * item.quantity,
        0
      )
    }
  },
  created() {
    this.getCurrentIOTAPrice()
  },
  methods: {
    addRow(e) {
      this.items.push({ description: '', quantity: 1, price: 0 })
    },
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.loading = true
          this.createPDF()
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    resetForm(formName) {
      this.$refs[formName].resetFields()
    },
    async createPDF() {
      if (process.browser) {
        const JsPDF = require('jspdf')

        const pdfName = 'test'
        const doc = new JsPDF({
          orientation: 'landscape'
        })

        console.log('doc', doc)
        // TODO: FIX THIS
        const source = window.document.getElementById('invoice')
        console.log('source', source)

        // doc.fromHTML(source, 15, 15, {})
        // TODO: END

        const text = `Hello World, IOTA Price: ${this.current_iota_price}`
        doc.text(text, 20, 20)

        doc.setProperties({
          title: 'shelli invoice',
          subject: 'PDF generated with shelli invoice',
          author: 'huhn',
          keywords: 'generated, javascript,jspdf',
          creator: 'Javascript jsPDF'
        })

        const ab = doc.output('arraybuffer')
        const hash = sha256(ab)
        console.log('hash', hash)

        try {
          const seed =
            'EIQLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWORL9D'
          const address =
            'EIDVKCDQAPYQOJEQTUWDMNYZKDUDBRNHJWV9VTKTCUUYQICLPFBETMYYVKEPFCXZE9EJZHFUWJZVEWUCWSGDUVMOYD'
          const provider = 'https://altnodes.devnet.iota.org'
          const depth = 3
          const minWeightMagnitude = 9 // 14 for Mainnet
          const tag = 'EINFACH9'
          const data = hash
          const bundle = {
            provider,
            data,
            tag,
            address,
            seed,
            depth,
            minWeightMagnitude
          }
          const retArr = await this.publish(bundle)
          console.log(`TX Hash=${retArr[0].hash}`)
          doc.save(pdfName + '.pdf')
        } catch (e) {
          console.log(`something went wrong ${e}`)
        }
        this.loading = false
      }
    },
    async publish(bundle) {
      const iota = composeAPI({
        provider: bundle.provider
      })
      const trytes = asciiToTrytes(bundle.data)
      // Array of transfers which defines transfer recipients and value transferred in IOTAs.
      const transfers = [
        {
          address: bundle.address,
          value: 0, // 1Ki
          tag: bundle.tag ? bundle.tag.toUpperCase() : '', // optional tag of `0-27` trytes
          message: trytes // optional message in trytes
        }
      ]

      let prepTrytes = null
      let ret = null
      try {
        prepTrytes = await iota.prepareTransfers(bundle.seed, transfers)
        ret = await iota.sendTrytes(
          prepTrytes,
          bundle.depth,
          bundle.minWeightMagnitude
        )
        return ret
      } catch (err) {
        console.log(`Could not establish a connection to the node ${err}`)
      }
    },
    getCurrentIOTAPrice() {
      axios.get('https://api.coingecko.com/api/v3/coins/iota').then((res) => {
        console.log('EUR', res)
        console.log('EUR', res.data.market_data.current_price.eur)
        this.current_iota_price = res.data.market_data.current_price.eur
      })
    }
  }
}
</script>
<style>
.invoice-box {
  max-width: 800px;
  margin: auto;
  padding: 30px;
  border: 1px solid #eee;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.15);
  font-size: 16px;
  line-height: 24px;
  font-family: 'Helvetica Neue', 'Helvetica', Helvetica, Arial, sans-serif;
  color: #555;
}

.invoice-box table {
  width: 100%;
  line-height: inherit;
  text-align: left;
}

.invoice-box table td {
  padding: 5px;
  vertical-align: top;
}

.invoice-box table tr td:nth-child(n + 2) {
  text-align: right;
}

.invoice-box table tr.top table td {
  padding-bottom: 20px;
}

.invoice-box table tr.top table td.title {
  font-size: 45px;
  line-height: 45px;
  color: #333;
}

.invoice-box table tr.information table td {
  padding-bottom: 40px;
}

.invoice-box table tr.heading td {
  background: #eee;
  border-bottom: 1px solid #ddd;
  font-weight: bold;
}

.invoice-box table tr.details td {
  padding-bottom: 20px;
}

.invoice-box table tr.item td {
  border-bottom: 1px solid #eee;
}

.invoice-box table tr.item.last td {
  border-bottom: none;
}

.invoice-box table tr.item input {
  padding-left: 5px;
}

.invoice-box table tr.item td:first-child input {
  margin-left: -5px;
  width: 100%;
}

.invoice-box table tr.total td:nth-child(2) {
  border-top: 2px solid #eee;
  font-weight: bold;
}

.invoice-box input[type='number'] {
  width: 60px;
}

@media only screen and (max-width: 600px) {
  .invoice-box table tr.top table td {
    width: 100%;
    display: block;
    text-align: center;
  }

  .invoice-box table tr.information table td {
    width: 100%;
    display: block;
    text-align: center;
  }
}

/** RTL **/
.rtl {
  direction: rtl;
  font-family: Tahoma, 'Helvetica Neue', 'Helvetica', Helvetica, Arial,
    sans-serif;
}

.rtl table {
  text-align: right;
}

.rtl table tr td:nth-child(2) {
  text-align: left;
}
</style>
