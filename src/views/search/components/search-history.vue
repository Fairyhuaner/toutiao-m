<template>
  <div class="search-history">
    <van-cell title="搜索历史">
      <div v-if="isDeleteShow">
        <span @click="$emit('empty-search-histories')">全部删除</span>
        &nbsp;
        <span @click="isDeleteShow = false">完成</span>
      </div>
      <van-icon name="delete" v-else @click="isDeleteShow = true" />
    </van-cell>
    <van-cell
      v-for="(item, index) in searchHistories"
      :key="index"
      :title="item"
      @click="onSearchItemClick(item, index)"
    >
      <van-icon name="close" v-show="isDeleteShow" />
    </van-cell>
  </div>
</template>

<script>
export default {
  name: 'SearchHistory',
  components: {},
  props: {
    searchHistories: {
      type: Array,
      required: true
    }
  },
  data () {
    return {
      isDeleteShow: false
    }
  },
  computed: {},
  watch: {},
  created () { },
  mounted () {
  },
  methods: {
    onSearchItemClick (item, index) {
      if (this.isDeleteShow) {
        // 删除状态
        this.$emit('del-search-histories', index)
      } else {
        // 点击跳转到搜索结果
        this.$emit('search', item)
      }
    }
  }
}
</script>

<style scoped lang="less"></style>
