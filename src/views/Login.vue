<template>
  <el-container>
    <el-header>
      <h1>Byerun</h1>
    </el-header>
    <el-main>
      <el-form v-if="showLoginForm">
        <el-form-item>
          <el-input v-model="phone" placeholder="请输入手机号"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input
            v-model="password"
            type="password"
            placeholder="请输入密码"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button
            type="primary"
            @click="LoginHandler"
            :loading="LoginLoading"
            >立即登录</el-button
          >
        </el-form-item>
        <el-form-item>
          <el-button type="default" @click="goToHome">返回主页</el-button>
        </el-form-item>
      </el-form>
      <el-form v-else-if="showDisclaimerForm" class="DisclaimerForm">
        <el-space direction="vertical">
          <el-text tag="b" size="large">免责声明</el-text>
          <el-text>为了您的健康，不提倡长期使用。</el-text>
          <el-text>使用本工具所产生的任何后果，用户需自行承担。</el-text>
          <el-text>本工具仅供学习交流使用，不得用于任何商业用途。</el-text>
          <el-link type="primary" :href="opensrcUrl" target="_blank"
            >开源地址</el-link
          >
        </el-space>
        <el-button
          type="primary"
          @click="
            showLoginForm = true;
            showDisclaimerForm = false;
          "
          >返回登录</el-button
        >
      </el-form>
      <el-form v-else>
        <el-form-item>
          <el-input
            v-model="phoneNum"
            placeholder="请输入手机号"
            class="phone-input"
          ></el-input>
          <el-button
            type="primary"
            @click="SendSMSHandler"
            :disabled="codeDisabled"
            :loading="SmsLoading"
            block
            class="sms-button"
            >{{ codeText }}</el-button
          >
        </el-form-item>
        <el-form-item>
          <el-input v-model="smsCode" placeholder="请输入验证码"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input
            v-model="newPassword"
            type="password"
            placeholder="请输入新密码"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button
            type="primary"
            @click="ResetPasswordHandler"
            :loading="ResetLoading"
            >提交</el-button
          >
        </el-form-item>
        <el-button type="text" @click="showLoginForm = true"
          >返回登录</el-button
        >
      </el-form>

      <div class="footer-links" v-if="showLoginForm">
        <el-link type="primary" target="_blank" @click="showDisclaimer"
          >免责声明</el-link
        >
        <el-link type="primary" @click="showLoginForm = false"
          >忘记密码</el-link
        >
      </div>
    </el-main>
  </el-container>
</template>

<script setup>
import "@/styles/login/index.css";
import { ref, watchEffect, onMounted } from "vue";
import { useRouter } from "vue-router";
import { ElMessage } from "element-plus";
import { useLogin, useSms, useResetPassword } from "@/hooks/login/";
import address from "@/services/address";

const opensrcUrl = address.opensrcURL;

const router = useRouter();

const showLoginForm = ref(true);
const showDisclaimerForm = ref(false);
const phone = ref("");
const password = ref("");
const phoneNum = ref("");
const newPassword = ref("");
const smsCode = ref("");

const { LoginLoading, isLoggedIn, fetchLogin, LoginState } = useLogin();
const { SmsLoading, codeText, codeDisabled, fetchSendSMS } = useSms();
const { ResetLoading, fetchResetPassword } = useResetPassword();

// 显示免责声明表单
const showDisclaimer = () => {
  showLoginForm.value = false;
  showDisclaimerForm.value = true;
};

const PhoneValid = () => {
  if (!/^1[3-9]\d{9}$/.test(phone.value || phoneNum.value)) {
    ElMessage.error("请输入正确的手机号");
    return false;
  } else {
    return true;
  }
};

const LoginHandler = () => {
  if (PhoneValid()) {
    fetchLogin(phone.value, password.value);
  }
};

const SendSMSHandler = () => {
  if (PhoneValid()) {
    fetchSendSMS(phoneNum.value);
  }
};

const ResetPasswordHandler = async () => {
  if (PhoneValid()) {
    const success = await fetchResetPassword(
      phoneNum.value,
      newPassword.value,
      smsCode.value
    );
    if (success) {
      phone.value = phoneNum.value;
      password.value = newPassword.value;
      showLoginForm.value = true;
    }
  }
};

const goToHome = () => {
  router.push("/");
};

// 路由跳转
const goToDashboard = () => {
  router.push("/dashboard");
};

// 监听登录状态变化，自动跳转
watchEffect(() => {
  if (isLoggedIn.value) {
    goToDashboard();
  }
});

// 页面加载完成后检查 token 和 userData
onMounted(async () => {
  const loginState = await LoginState();
  if (loginState) {
    goToDashboard();
  }
});
</script>

<style scoped></style>
