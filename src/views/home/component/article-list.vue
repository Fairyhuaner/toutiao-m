<template>
  <div class="article-list">
    <van-pull-refresh
      v-model="isLoading"
      @refresh="onRefresh"
      :success-text="refreshSuccessText"
      success-duration="1000"
    >
      <van-list
        v-model="loading"
        :finished="finished"
        finished-text="没有更多了"
        :error.sync="error"
        error-text="请求失败，点击重新加载"
        @load="onLoad"
      >
        <ArticleItem
          v-for="(article, index) in list"
          :key="index"
          :article="article"
        />
        <!-- <van-cell
          v-for="(item, index) in list"
          :key="index"
          :title="item.title"
        /> -->
      </van-list>
    </van-pull-refresh>
  </div>
</template>

<script>
import { getArticles } from '@/api/article'
import ArticleItem from '@/components/ArticleItem'
export default {
  props: {
    channel: {
      type: Object,
      required: true
    }
  },
  name: 'ArticleList',
  created () { },
  data () {
    return {
      list: [], // 存储列表数据
      loading: false, // 控制加载中loading状态
      finished: false, // 控制数据加载结束的状态
      timestamp: null, // 请求获取下一页数据的事件戳
      error: false, // 列表加载失败的提示状态
      isLoading: false, // 控制下拉刷新的状态
      refreshSuccessText: '' // 下拉刷新提示
    }
  },
  methods: {
    // 初始化或滚动到底部时候会触发函数
    async onLoad () {
      try {
        const { data } = await getArticles({
          channel_id: this.channel.id, // 频道id
          timestamp: this.timestamp || Date.now() //
        })
        // 把获取的数据解构出来
        const { results } = data.data
        // 不能直接给list 赋值  需要用push添加 ...是合并
        this.list.push(...results)
        // 加载完数据之后 用false结束
        this.loading = false
        if (results.length) {
          // 有数据的时候
          this.timestamp = data.data.pre_timestamp
        } else {
          // 没有数据的时候
          this.finished = true
        }
      } catch (err) {
        // vant 组件里面list组件自带的错误提示  设置为为true是显示错误状态
        this.error = true

        // 请求失败了 loading也需要关闭 不用在往下加载了
        this.loading = false
      }
    },
    // 当下拉刷新的时候会调用函数
    async onRefresh () {
      try {
        //  请求获取数据
        const { data } = await getArticles({
          channel_id: this.channel.id, // 频道id
          timestamp: this.timestamp || Date.now()
        })
        const { results } = data.data
        // 将获取的数据追加到列表的顶部 unshift
        this.list.unshift(...results)
        // 将数据放到列表的顶部
        // 关闭下拉刷新的loading状态
        this.isLoading = false

        this.refreshSuccessText = `刷新成功，更新了${results.length}数据`
      } catch (err) {
        this.refreshSuccessText = '刷新失败'
        this.isLoading = false
      }
    }
  },
  computed: {},
  watch: {},
  filters: {},
  components: { ArticleItem }
}
</script>

<style scoped lang='less'>
.article-list {
  height: 79vh;
  // overflow: hidden;
}
</style>
