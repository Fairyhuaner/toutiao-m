<template>
  <div class="home-container">
    <!-- 导航栏 -->
    <van-nav-bar class="page-nav-bar" fixed>
      <van-button
        class="search-btn"
        slot="title"
        type="info"
        size="small"
        round
        icon="search"
        to="/search"
        >搜索</van-button
      >
    </van-nav-bar>
    <!-- /导航栏 -->

    <!-- 频道列表 -->
    <!--
通过v-model绑定当前激活标签对应的索引值，默认情况下启用第一个标签
通过animated属性可以开启切换标签内容时的转场动画
通过swipeable属性可以开启滑动切换标签页
-->
    <van-tabs class="channel-tabs" v-model="active" animated swipeable>
      <van-tab
        :title="channel.name"
        v-for="channel in channels"
        :key="channel.id"
      >
        <div class="wrapper">
          <ArticleList
            v-for="(item, index) in channels"
            :key="index"
            :channel="item"
          >
          </ArticleList>
        </div>
      </van-tab>

      <div slot="nav-right" class="placeholder"></div>
      <div slot="nav-right" class="hamburger-btn">
        <i
          class="toutiao toutiao-gengduo"
          @click="isChennelEditShow = true"
        ></i>
      </div>
    </van-tabs>
    <!-- 频道列表 -->

    <!--  频道弹出层 -->
    <van-popup
      v-model="isChennelEditShow"
      closeable
      position="bottom"
      :style="{ height: '100%' }"
      close-icon-position="top-left"
    >
      <ChennelEdit
        :myChannels="channels"
        :active="active"
        @updata-active="onUpdataActive"
      ></ChennelEdit>
    </van-popup>
    <!--  频道弹出层 -->
  </div>
</template>

<script>
import { getUserChannels } from '@/api/user'
import ArticleList from '@/views/home/component/article-list.vue'
import ChennelEdit from './component/channel-edit.vue'
export default {
  name: 'HomeIndex',
  created () {
    this.loadChannels()
  },
  data () {
    return {
      active: 0,
      channels: [], // 频道列表
      isChennelEditShow: false // 控制编辑频道弹出层的显示状态
    }
  },
  methods: {
    async loadChannels () {
      try {
        const { data } = await getUserChannels()
        this.channels = data.data.channels
      } catch (err) {
        this.$toast('获取数据失败')
      }
    },
    onUpdataActive (index, isChennelEditShow = true) {
      this.active = index
      this.isChennelEditShow = false
    }
  },
  computed: {},
  watch: {},
  filters: {},
  components: { ArticleList, ChennelEdit }
}
</script>

<style scoped lang='less'>
.home-container {
  padding-top: 174px;
  padding-bottom: 100px;
  /deep/.van-nav-bar__title {
    max-width: unset;
  }
  .search-btn {
    width: 555px;
    height: 64px;
    background-color: #5babfb;
    border: none;
    font-size: 28px;
    .van-icon {
      font-size: 32px;
    }
  }
}
/deep/ .channel-tabs {
  .van-tab {
    border-right: 1px solid #edeff3;
    min-width: 200px;
    height: 80px;
    .van-tab__text {
      font-size: 27px;
      color: #777777;
    }
  }

  .van-tab--active .van-tab__text {
    color: #333 !important;
  }
  .van-tabs__line {
    width: 31px !important;
    height: 6px;
    background: rgba(50, 150, 250, 1);
    border-radius: 3px;
    bottom: 8px;
  }
  .van-tabs__nav {
    padding: 0;
  }
  .placeholder {
    flex-shrink: 0;
    width: 66px;
    height: 82px;
  }
  .hamburger-btn {
    position: fixed;
    right: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 66px;
    height: 82px;
    background-color: #fff;
    opacity: 0.902;
    i.toutiao {
      font-size: 33px;
    }
    &:before {
      content: "";
      position: absolute;
      left: 0;
      width: 1px;
      height: 100%;
      background-image: url(~@/assets/logo.png);
      background-size: contain;
    }
  }
  .van-tabs__wrap {
    position: fixed;
    top: 92px;
    z-index: 1;
    left: 0;
    right: 0;
    height: 82px;
  }
}
.wrapper {
  height: 100vh;
  box-sizing: border-box;
  // padding-top: 20px;
  padding-bottom: 100px;
  .article-list {
    height: calc(100vh - 272px);
    overflow: auto;
  }
}
</style>
