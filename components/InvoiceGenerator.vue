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

                <td>
                  Invoice #:
                  <input v-model="form.invoice_number" />
                  <br />Created:
                  <input v-model="form.invoice_created_at" />
                  <br />Due:
                  <input v-model="form.invoice_due_at" />
                </td>
              </tr>
            </table>
          </td>
        </tr>

        <tr class="information">
          <td colspan="4">
            <table>
              <tr>
                <td>
                  <p>
                    <input v-model="form.company_name" />
                  </p>
                  <p>
                    <input v-model="form.address" />
                  </p>
                  <p>
                    <input v-model="form.zip_code" />
                    <input v-model="form.city" />
                  </p>
                </td>

                <td>
                  <p>
                    <input v-model="form.receiver.company_name" />
                  </p>
                  <p>
                    <input v-model="form.receiver.address" />
                  </p>
                  <p>
                    <input v-model="form.receiver.zip_code" />
                    <input v-model="form.receiver.city" />
                  </p>
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
          <td>
            <input v-model="item.description" />
          </td>
          <td><input v-model="item.price" type="number" /> €</td>
          <td>
            <input v-model="item.quantity" type="number" />
          </td>
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
      <el-button @click="resetForm('form')">Reset</el-button>
      <el-button
        @click="submitForm('form')"
        v-loading.fullscreen.lock="loading"
        type="primary"
        >Create invoice</el-button
      >
    </el-form-item>
    <div id="result" v-loading="loading">
      <h2>Your transaction hash</h2>
      <p>
        Share this hash with your invoice PDF document, to proof it with the
        invoice verifier.
      </p>
      <p v-if="tx_hash">{{ tx_hash }}</p>
      <el-alert
        v-else
        :closable="false"
        title="Create an invoice first."
        type="warning"
      ></el-alert>
    </div>
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
      form: {
        company_name: 'shelli, Inc',
        address: '42 Big Road',
        zip_code: '39527',
        city: 'Sunnyville, CA',
        country: 'USA',
        invoice_number: 342,
        invoice_created_at: 'February 29, 2020',
        invoice_due_at: 'March 9, 2020',
        receiver: {
          company_name: 'John Doe Corp',
          address: '4320  Mount Street',
          zip_code: '48607',
          city: 'Saginaw',
          country: 'USA'
        }
      },
      rules: {},
      current_iota_price: null,
      items: [
        { description: 'Website concept', quantity: 1, price: 900 },
        { description: 'Website design', quantity: 1, price: 3000 },
        { description: 'Website deployment', quantity: 1, price: 300 }
      ],
      tx_hash: ''
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
      console.log('button pressed')
      this.loading = true
      this.$refs[formName].validate((valid) => {
        if (valid) {
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
          orientation: 'portrait'
        })

        doc.setProperties({
          title: 'shelli invoice',
          subject: 'PDF generated with shelli invoice',
          author: 'huhn',
          keywords: 'generated, javascript,jspdf',
          creator: 'Javascript jsPDF'
        })
        const fontSizes = {
          HeadTitleFontSize: 20,
          Head2TitleFontSize: 18,
          SubTitleFontSize: 16,
          NormalFontSize: 15,
          SmallFontSize: 10
        }

        doc.setFont('open-sans')
        doc.setFontSize(fontSizes.NormalFontSize)

        // Invoice Information
        doc.text(`Invoice #${this.form.invoice_number}`, 190, 36, 'right')
        doc.text(`Created: ${this.form.invoice_created_at}`, 190, 42, 'right')
        doc.text(`Due: ${this.form.invoice_due_at}`, 190, 48, 'right')

        // Anschrift
        doc.text(`${this.form.company_name}`, 20, 68)
        doc.text(`${this.form.address}`, 20, 74)
        doc.text(`${this.form.zip_code} ${this.form.city}`, 20, 80)

        doc.text(`${this.form.receiver.company_name}`, 190, 68, 'right')
        doc.text(`${this.form.receiver.address}`, 190, 74, 'right')
        doc.text(
          `${this.form.receiver.zip_code} ${this.form.receiver.city}`,
          190,
          80,
          'right'
        )

        // Logo bg
        const img = new Image()
        img.src = require('@/assets/shelli-bg.png')
        doc.addImage(img, 'PNG', 0, 0)

        // Logo bg
        img.src = require('@/assets/logo.png')
        doc.addImage(img, 'PNG', 20, 30, 25, 25)

        // Payment Method and IOTA Live Data
        doc.setFontSize(fontSizes.Head2TitleFontSize)
        doc.text(`Payment Method`, 20, 120)
        doc.text(`MIOTA Price`, 190, 120, 'right')

        doc.setFontSize(fontSizes.NormalFontSize)
        doc.text(`IOTA`, 20, 130)
        doc.text(`${this.current_iota_price} €`, 190, 130, 'right')

        // Items Header
        doc.setFontSize(fontSizes.Head2TitleFontSize)
        doc.text(`Item`, 20, 150)
        doc.text(`Unit Cost`, 120, 150, 'right')
        doc.text(`Quantity`, 150, 150, 'right')
        doc.text(`Price`, 190, 150, 'right')

        // Items Header
        doc.setFontSize(fontSizes.NormalFontSize)
        let x = 0
        this.items.forEach((item) => {
          doc.text(`${item.description}`, 20, 160 + x)
          doc.text(`${item.price}`, 120, 160 + x, 'right')
          doc.text(`${item.quantity}`, 150, 160 + x, 'right')
          doc.text(`${item.price * item.quantity} €`, 190, 160 + x, 'right')
          x = x + 10
        })

        // Total
        doc.setFontSize(fontSizes.Head2TitleFontSize)
        doc.text(`Total: ${this.total} €`, 190, 160 + x + 10, 'right')

        // Footer
        doc.setFontSize(fontSizes.SmallFontSize)
        doc.text(`shelli.io - Accounting as a Service`, 20, 290)

        const ab = doc.output('arraybuffer')
        const hash = sha256(ab)
        console.log('hash', hash)

        try {
          const seed =
            'EIQLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWORL9D'
          const address =
            'EIDVKCDQAPYQOJEQTUWDMNYZKDUDBRNHJWV9VTKTCUUYQICLPFBETMYYVKEPFCXZE9EJZHFUWJZVEWUCWSGDUVMOYD'
          const provider = 'https://nodes.devnet.thetangle.org:443'
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
          this.tx_hash = retArr[0].hash
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
        this.current_iota_price = res.data.market_data.current_price.eur
      })
    }
  }
}
</script>
<style lang="scss">
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

.el-form-item__content {
  margin-top: 50px;
}

#result {
  padding: 50px 20px;
  margin-top: 50px;
  p {
    padding: 5px;
    word-break: break-all;
  }
}
</style>
