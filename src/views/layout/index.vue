<template>
  <div class="layout-container">
    <!-- 子路由 出口 -->
    <router-view></router-view>
    <!-- 子路由 出口 -->

    <!-- 标签导航栏 -->
    <!-- router就是tabber 开启路由模式 -->
    <van-tabbar class="layout-babber" route>
      <van-tabbar-item to="/home">
        <i slot="icon" class="toutiao toutiao-shouye"></i>
        <span class="text">首页</span>
      </van-tabbar-item>
      <van-tabbar-item to="/qa">
        <i slot="icon" class="toutiao toutiao-wenda"></i>
        <span class="text">问答</span>
      </van-tabbar-item>
      <van-tabbar-item to="/video">
        <i slot="icon" class="toutiao toutiao-shipin"></i>
        <span class="text">视频</span>
      </van-tabbar-item>
      <van-tabbar-item to="/my">
        <i slot="icon" class="toutiao toutiao-wode"></i>
        <span class="text">{{$store.state.user? '我的':'未登录'}}</span>
      </van-tabbar-item>
    </van-tabbar>
    <!-- 标签导航栏 -->
  </div>
</template>

<script>

export default {
  name: 'LayoutIndex',
  created () { },
  data () {
    return {

    }
  },
  methods: {},
  computed: {},
  watch: {},
  filters: {},
  components: {}
}
</script>

<style scoped lang="less">
.layout-container {
  .layout-babber {
    i.toutiao {
      font-size: 40px;
    }
    span.text {
      font-size: 20px;
    }
  }
}
</style>
