<template>
  <el-form ref="captchaForm" class="login-content accept-content" :rules="rules" :model="form">
    <div class="accept-container">
      <el-button class="plain-button backButton" icon="arrow-left" @click="back"></el-button>
      <header class="accept-header">
        <div class="accept-title">{{header.title}}</div>
        <div class="accept-subtitle">{{header.subtitle}}</div>
      </header>
      <div class="login-flow">
        <el-form-item class="flow-account accept-account">
          <el-input :value="`${form.code}${form.username}`" class="account" disabled></el-input>
        </el-form-item>
        <el-form-item class="flow-nickname" prop="captcha">
          <el-input v-model="form.captcha" placeholder="请输入 6 位短信验证码" class="nickname">
            <span slot="append" class="await-time" v-if="awaitCaptcha">{{awaitCaptcha}} 秒后可重发</span>
            <el-button slot="append" class="switch-type" @click="accept" v-else>重新获取短信验证码</el-button>
          </el-input>
        </el-form-item>
        <div class="login-options">
          <el-button class="forget-pass" @click="back">更换登录方式</el-button>
        </div>
        <footer class="accept-footer-container">
          <el-button class="submit enter" @click="signup">进入知书</el-button>
        </footer>
      </div>
    </div>
  </el-form>
</template>

<script>
  import { requireSMS, signup } from '@/api'
  import { userCheckRules } from '@/utils/validate'

  export default {
    name: 'captcha-card',
    props: {
      userform: Object,
      back: Function
    },
    data () {
      return {
        header: {
          title: '验证手机',
          subtitle: '请输入你收到的 6 位数验证码'
        },
        form: { ...this.userform },
        awaitCaptcha: 0,
        rules: userCheckRules
      }
    },
    mounted () {
      this.accept()
    },
    methods: {
      async accept () {
        requireSMS(this.form)
        this.awaitCaptcha = 59
        let timeout = setInterval(() => {
          if (this.awaitCaptcha <= 0) {
            clearInterval(timeout)
          } else {
            this.awaitCaptcha--
          }
        }, 1000)
      },
      async signup () {
        this.$refs.captchaForm.validate(valid => {
          if (!valid) throw new Error('signup form not validated')
        })
        let user = await signup(this.form)
        this.$emit('signed', user)
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus" scoped>
  @import './signup-card'
</style>
