<template>
  <view class="container">
    <view :style="'height:' + (statusBarHeight + 5) + 'px;'"></view>
    <view class="form">
      <view class="textlogin">
        <text>找回密码</text>
      </view>

      <input
        type="text"
        placeholder="请输入手机号"
        class="input"
        @input="updateField('account', $event)"
      />
      <input
        type="password"
        placeholder="请确认密码"
        class="input"
        @input="updateField('password1', $event)"
      />
      <input
        type="password"
        placeholder="请在此确认密码"
        class="input"
        @input="updateField('password2', $event)"
      />
      <view class="button1">
        <input
          type="text"
          placeholder="请输入验证码"
          class="input button2"
          @input="updateField('code', $event)"
        />
        <!-- <button class="button3" @click="requestVerificationCode">获取</button> -->
        <view
          @click.stop="requestVerificationCode"
          :disabled="isCodeSent"
          :class="{ 'code-button': true, 'disabled-button': isCodeSent }"
        >
          {{ isCodeSent ? `${countdown}秒` : '获取验证码' }}
        </view>
      </view>
      <button class="login-button" @click="submitResetPassword">完成</button>
    </view>

    <view class="footer"></view>
  </view>
</template>

<script setup>
import { ref } from 'vue'
import { fetchFindPassword, fetchVerificationCode } from '@/utils/api'
import { onLoad } from '@dcloudio/uni-app'
const statusBarHeight = ref()
onLoad(() => {
  statusBarHeight.value = getApp().globalData.top
})
const account = ref('')
const password1 = ref('')
const password2 = ref('')
const code = ref('')
const isCodeSent = ref(false)
const countdown = ref(60)
let timer = null

const updateField = (field, event) => {
  if (field === 'account') account.value = event.detail.value
  if (field === 'password1') password1.value = event.detail.value
  if (field === 'password2') password2.value = event.detail.value
  if (field === 'code') code.value = event.detail.value
}

const isValidPhoneNumber = (phoneNumber) => phoneNumber.length === 11

const requestVerificationCode = async () => {
  const phonePattern = /^[1][3-9][0-9]{9}$/

  if (!phonePattern.test(account.value)) {
    return uni.showToast({ title: '手机号格式不正确', icon: 'none' })
  }

  uni.showLoading({
    title: '发送中...',
  })
  try {
    const response = await fetchVerificationCode({
      mobile: account.value,
      event: 'resetpwd',
    })
    console.log(response)

    if (response.code === 1) {
      isCodeSent.value = true
      startCountdown()
      uni.showToast({
        title: response.msg || '验证码发送成功',
        icon: 'success', // 提示图标类型
        duration: 2000, // 持续时间
      })
    } else {
      uni.showToast({
        title: response.msg || '发送验证码失败，请重试',
        icon: 'none', // 提示图标类型
        duration: 2000, // 持续时间
      })
    }
  } catch (error) {
    uni.showToast({
      title: '发送验证码失败，请重试',
      icon: 'none', // 提示图标类型
      duration: 2000, // 持续时间
    })
    console.error('发送验证码错误:', error)
  } finally {
    uni.hideLoading()
  }
  // try {
  //   await fetchVerificationCode({
  //     mobile: account.value,
  //     event: 'changemobile',
  //   })
  //   isCodeSent.value = true
  //   startCountdown()
  //   showToast('验证码发送成功', 'success')
  // } catch (error) {
  //   showToast('发送验证码失败，请重试', 'error')
  //   console.error('发送验证码错误:', error)
  // } finally {
  //   uni.hideLoading()
  // }
}
const startCountdown = () => {
  countdown.value = 60
  clearInterval(timer)
  timer = setInterval(() => {
    countdown.value--
    if (countdown.value <= 0) {
      clearInterval(timer)
      isCodeSent.value = false
    }
  }, 1000)
}
const submitResetPassword = async () => {
  if (!account.value || !password1.value || !password2.value || !code.value) {
    return showError('请完整填写信息')
  }

  if (password1.value !== password2.value) {
    return showError('两次密码输入不一致')
  }

  try {
    const response = await fetchFindPassword({
      mobile: account.value,
      newpassword: password2.value,
      captcha: code.value,
      type: 'mobile',
    })

    if (response.code === 1) {
      uni.showToast({
        title: response.msg || '密码重置成功',
        icon: 'success',
        duration: 2000,
      })
      uni.redirectTo({ url: '/pages/login/login' })
    }
  } catch (error) {
    showError('网络异常，请稍后再试')
    console.error('重置密码错误:', error)
  }
}

const showError = (message) => {
  uni.showToast({
    title: message,
    icon: 'error',
    duration: 2000,
  })
}
</script>

<style scoped>
body {
  overflow: hidden;
}
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-around;
  margin-top: 200rpx;
  background-color: #fff;
  height: auto !important;
}

.form {
  width: 80%;
}

.input {
  padding: 10px;
  margin-bottom: 15px;
  border-radius: 50px;
  background-color: #eeeeee;
  font-size: 14px;
}

.login-button {
  width: 100%;
  padding: 2px;
  border-radius: 50px;
  background-color: #666666;
  color: #fff;
  text-align: center;
  font-size: 16px;
  margin-top: 100rpx;
}

.textlogin {
  margin-bottom: 24px;
  font-weight: 700;
}

.footer {
  margin-top: 20px;
  text-align: center;
  font-size: 12px;
  color: #888;
}

.button1 {
  display: flex;

  align-items: center;
  margin-bottom: 30rpx;
}

.button2 {
  margin-bottom: 0;
}

.button3 {
  width: 30%;
  margin-right: 0;
  border-radius: 60rpx;
  font-size: 14px;
  border: 0.5px #555 solid;
}
.code-button {
  width: 110px;
  height: 32px;
  border-radius: 100px;

  font-size: 12px;
  text-align: center;
  line-height: 32px;
}

.disabled-button {
  pointer-events: none;
}
</style>
