<template>
  <el-form
    ref="form"
    :model="form"
    :rules="rules"
    status-icon
    label-width="120px"
    class="form"
  >
    <h3>Your Company Information</h3>
    <el-form-item label="Company name" prop="company_name">
      <el-input
        v-model="form.company_name"
        type="text"
        autocomplete="off"
      ></el-input>
    </el-form-item>
    <el-form-item label="Tax number" prop="tax_number">
      <el-input
        v-model="form.tax_number"
        type="text"
        autocomplete="off"
      ></el-input>
    </el-form-item>
    <h3>Costumer Information</h3>
    <el-form-item label="First name" prop="first_name">
      <el-input
        v-model="form.customer.first_name"
        type="text"
        autocomplete="off"
      ></el-input>
    </el-form-item>
    <el-form-item label="Last name" prop="last_name">
      <el-input
        v-model="form.customer.last_name"
        type="text"
        autocomplete="off"
      ></el-input>
    </el-form-item>
    <h3>IOTA Payment Information</h3>
    <el-form-item label="Transaction" prop="tx">
      <el-input v-model="form.tx" type="text" autocomplete="off"></el-input>
    </el-form-item>
    <el-form-item>
      <el-button @click="submitForm('ruleForm')" type="primary"
        >Create invoice</el-button
      >
      <el-button @click="resetForm('ruleForm')">Reset</el-button>
    </el-form-item>
  </el-form>
</template>
<script>
export default {
  data() {
    return {
      form: {
        company_name: '',
        tax_number: '',
        customer: {
          first_name: '',
          last_name: ''
        },
        tx: ''
      },
      rules: {}
    }
  },
  methods: {
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          alert('submit!')
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    resetForm(formName) {
      this.$refs[formName].resetFields()
    }
  }
}
</script>
<style></style>
