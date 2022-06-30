<template>
  <div class="login-container">
    <!-- 导航栏 -->
    <van-nav-bar title="登录" class="page-nav-bar">
      <van-icon slot="left" name="cross" @click="$router.back()" />
    </van-nav-bar>
    <!-- 导航栏 -->

    <!-- 登录表单 -->
    <van-form @submit="onSubmit" ref="loginForm">
      <van-field
        name="mobile"
        placeholder="请输入手机号"
        v-model="user.mobile"
        :rules="userFormRules.mobile"
        type="number"
        maxlength="11"
      >
        <i slot="left-icon" class="toutiao toutiao-shouji"></i>
      </van-field>
      <van-field
        name="code"
        placeholder="请输入验证码"
        v-model="user.code"
        :rules="userFormRules.code"
        type="number"
        maxlength="11"
      >
        <i slot="left-icon" class="toutiao toutiao-yanzhengma"></i>
        <template #button>
          <!--
            time：验证码的倒计时时间
           -->
          <van-count-down
            :time="1000 * 3"
            format="ss s"
            v-if="isCountDownShow"
            @finish="isCountDownShow = false"
          />
          <van-button
            v-else
            class="send-sms-btn"
            round
            size="small"
            type="default"
            @click="onSendSms"
            native-type="button"
            >发送验证码</van-button
          >
        </template>
      </van-field>
      <div class="login-btn-wrap">
        <van-button class="login-btn" block type="info" native-type="submit">
          登录
        </van-button>
      </div>
    </van-form>
    <!-- 登录表单 -->
  </div>
</template>

<script>
import { login, sendSms } from '@/api/user'
export default {
  name: 'LoginPage',
  components: {},
  props: {},
  data () {
    return {
      user: {
        mobile: '13911111111', // 手机号
        code: '246810' // 手机号
      },
      userFormRules: {
        mobile: [{
          required: true,
          message: '111'
        },
        {
          pattern: /^1[3|5|7|8]\d{9}$/,
          message: '手机号格式错误'
        }],
        code: [{
          required: true,
          message: '111'
        },
        {
          pattern: /^\d{6}$/,
          message: '验证错误'
        }]
      },
      isCountDownShow: false // 是否展示验证码倒计时
    }
  },
  computed: {},
  watch: {},
  created () { },
  mounted () { },
  methods: {
    async onSubmit () {
      // 1.获取表单数据
      const user = this.user
      // 2.表单验证
      // loading 转圈圈提示
      this.$toast.loading({
        duration: 0, // 持续展示 toast
        message: '加载中...',
        forbidClick: true // 是否禁止背景点击
      })
      // 3.提交表单请求登录
      try {
        const { data } = await login(user)

        this.$store.commit('setUser', data.data)
        this.$toast.success('登录成功')
        // 登录成功后跳转到原来的页面 不用一直转圈
        // back方式不严谨
        this.$router.back()
      } catch (err) {
        if (err.response.status === 400) {
          this.$toast.fail('手机号或验证码错误')
        } else {
          this.$toast.fail('登录失败，请稍后重试')
        }
      }
      // 4.根据请求响应结果处理后续操作
    },
    async onSendSms () {
      // 1.点击登录之后验证
      try {
        await this.$refs.loginForm.validate('mobile')

        console.log('验证通过')
      } catch (err) {
        return console.log('验证失败', err)
      }
      // 2.如果说验证通过我们要显示倒计时
      this.isCountDownShow = true
      // 3.请求发送验证码  用接口
      try {
        await sendSms(this.user.mobile)
        this.$toast('发送成功')
      } catch (err) {
        // 发送失败的时候把验证码倒计时关闭
        this.isCountDownShow = false
        if (err.response.status === 429) {
          this.$toast('发送太频繁了，请稍后再试')
        } else {
          this.$toast('发送失败')
        }
      }
    }

  }
}
</script>
<style scoped lang="less">
.login-container {
  .toutiao {
    font-size: 37px;
  }
  /deep/ .send-sms-btn {
    width: 158px;
    height: 46px;
    line-height: 46px;
    background-color: #ededed;
    font-size: 22px;
    color: #666;
  }
  .login-btn-wrap {
    padding: 53px 33px;
    .login-btn {
      background-color: #6db4fb;
      border: none;
    }
  }
}
</style>
