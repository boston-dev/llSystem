<template>
  <div>
    <p style="padding: 16px; text-align: center">提交提现订单</p>
    <el-input v-model.trim="apiUrl" placeholder="apiUrl" clearable></el-input>
    <el-input v-model.trim="token" placeholder="token" clearable></el-input>
    <el-form
      :model="form"
      :rules="rules"
      ref="form"
      label-width="150px"
      size="small"
    >
      <el-form-item label="金额" prop="amount">
        <el-input
          v-model.trim="form.amount"
          placeholder="请输入金额"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item label="收款人账号" prop="receiptAccountNo">
        <el-input
          v-model.trim="form.receiptAccountNo"
          placeholder="请输入收款人账号"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item label="用户姓名" prop="nameSurname">
        <el-input
          v-model.trim="form.nameSurname"
          placeholder="请输入用户姓名"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item label="收款人名称" prop="receiptAccountName">
        <el-input
          v-model.trim="form.receiptAccountName"
          placeholder="请输入收款人名称"
          clearable
        ></el-input>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="submitForm('form')"
          >提交订单</el-button
        >
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import axios from "axios";
import CryptoJS from "crypto-js";
import qs from "qs";
export default {
  data() {
    const validatePayee = (rule, value, callback) => {
      console.log(!this.form.nameSurname, !this.form.receiptAccountName);
      if (!this.form.nameSurname && !this.form.receiptAccountName) {
        callback(new Error("请至少填写一个收款名"));
      } else {
        callback();
      }
    };
    return {
      apiUrl: localStorage.getItem("apiUrl") || "",
      token: localStorage.getItem("apiUrl") || "", // 替换成实际的token
      form: {
        amount: "",
        nameSurname: "",
        receiptAccountName: "",
        receiptAccountNo: "",
      },
      rules: {
        amount: [
          { required: true, message: "请输入金额", trigger: "blur" },
          {
            pattern: /^(0|[1-9]\d*)(\.\d{2})?$/,
            message: "金额必须为数字，整数或保留两位小数",
            trigger: "blur",
          },
        ],
        nameSurname: [{ validator: validatePayee, trigger: "blur" }],
        receiptAccountName: [{ validator: validatePayee, trigger: "blur" }],
        receiptAccountNo: [
          {
            required: true,
            message: "请输入收款人账号",
            trigger: "blur",
          },
        ],
      },
    };
  },
  methods: {
    generateSignature(data) {
      // 按照签名规则构造待签名字符串
      const sortedKeys = Object.keys(data).sort();
      let concatenatedString = sortedKeys
        .map((key) => `${key}=${data[key]}`)
        .join("&");
      concatenatedString += this.token;

      // 使用 SHA-256 计算签名
      const hash = CryptoJS.HmacSHA256(concatenatedString, "");
      const signature = hash.toString(CryptoJS.enc.Hex);

      return signature;
    },
    vaidForm() {
      return new Promise((resolve) => {
        this.$refs.form.validate((valid) => {
          if (valid) {
            resolve(valid);
          } else {
            resolve(false);
          }
        });
      });
    },
    async submitForm(formName) {
      if (!this.apiUrl || !this.token) {
        alert("请填写 apiUrl 和 token");
        return;
      }
      //保存 apiUrl token
      localStorage.setItem("apiUrl", this.apiUrl);
      localStorage.setItem("token", this.token);
      const status = await this.vaidForm();
      console.log("表单验证结果:", status);
      if (!status) {
        return;
      }

      // 准备请求参数
      const withdrawalData = {
        ...this.form,
      };
      for (const key in withdrawalData) {
        if (withdrawalData[key] === "") {
          delete withdrawalData[key];
        }
      }
      // 生成签名
      const signature = this.generateSignature(withdrawalData);
      console.log("签名:", signature);
      // 构造请求数据
      const requestData = {
        ...withdrawalData,
        signature: signature, // 将签名添加到请求数据中
      };
      const formData = qs.stringify(requestData);
      console.log("请求数据:", formData);
      // 发送 POST 请求
      const response = await axios.post(this.apiUrl, formData, {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
      });
      console.log("提交结果:", response.data);
      // 提交成功后清空表单数据
      this.$refs[formName].resetFields();
    },
  },
};
</script>

<style>
/* 可以在这里添加样式 */
</style>
