<template>
  <div class="channel-edit">
    <van-cell title="我的频道" :border="false">
      <van-button
        size="mini"
        round
        type="danger"
        plain
        @click="isEdit = !isEdit"
        >{{ isEdit ? "完成" : "编辑" }}</van-button
      >
    </van-cell>
    <van-grid :gutter="10" class="my-grid">
      <van-grid-item
        class="grid-item"
        v-for="(channel, index) in myChannelsList"
        :key="index"
        @click="onMyChannelClick(channel, index)"
      >
        <!-- 编辑按钮显示隐藏 -->
        <van-icon
          v-show="isEdit && !fiexdChannels.includes(channel.id)"
          slot="icon"
          name="clear"
        ></van-icon>
        <span slot="text" :class="{ active: index === active }" class="text">{{
          channel.name
        }}</span>
      </van-grid-item>
    </van-grid>
    <van-cell title="频道推荐" :border="false"></van-cell>
    <van-grid :gutter="10" class="recommend-grid">
      <van-grid-item
        class="grid-item"
        v-for="(channel, index) in recommendChannels"
        :key="index"
        :text="channel.name"
        @click="onAddChannel(channel)"
      />
    </van-grid>
  </div>
</template>

<script>
import {
  getAllChannels,
  addUserChannel,
  deleteUserChannel
} from '@/api/channel'
import { mapState } from 'vuex'
import { setItem } from '@/utils/storage'

export default {
  created () {
    this.loadAllChannels()
    this.myChannelsList = this.myChannels
  },
  name: 'ChannelEdit',
  components: {},
  props: {
    myChannels: {
      type: [Array, Object],
      required: true
    },
    active: {
      type: Number,
      required: true
    }
  },
  data () {
    return {
      allChannels: [], // 所有频道
      myChannelsList: [],
      isEdit: false, // 控制编辑状态的显示
      fiexdChannels: [0] // 固定频道不能删除(推荐)

    }
  },
  computed: {
    ...mapState(['user']),
    // eslint-disable-next-line vue/return-in-computed-property
    recommendChannels () {
      return this.allChannels.filter(channel => {
        return !this.myChannels.find(myChannel => { return myChannel.id === channel.id })
      })
    }
    // recommendChannels () {
    //   const channels = []
    //   this.allChannels.forEach(channel => {
    //     // find遍历foreach遍历过的数组  找到满足条件的返回
    //     const ret = this.myChannels.find(myChannel => {
    //       return myChannel.id === channel.id
    //     })
    //     if (!ret) {
    //       channels.push(channel)
    //     }
    //   })
    //   // 计算属性里面要return出去
    //   return channels
    // }
  },
  watch: {},

  mounted () {
  },
  methods: {
    async loadAllChannels () {
      try {
        const { data } = await getAllChannels()
        this.allChannels = data.data.channels
      } catch (err) {
        this.$toast('获取失败')
      }
    },
    async onAddChannel (channel) {
      this.myChannelsList.push(channel)

      // 数据持久化处理
      if (this.user) {
        try {
          // 已登录,把数据请求接口放到线上
          await addUserChannel({
            id: channel.id, // 频道id
            seq: this.myChannels.length // 序号
          })
        } catch (err) {
          this.$toast('保存失败')
        }
      } else {
        // 未登录，把数据存储到本地
        setItem('TOUTIAO_CHANNELS', this.myChannelsList)
      }
    },
    onMyChannelClick (channel, index) {
      // 因为我们之前自定义了一个属性状态isEdit 用来判断编辑还是完成
      if (this.isEdit) {
        if (this.fiexdChannels.includes(index)) {
          return
        }
        // 如果是编辑状态执行删除
        this.myChannelsList.splice(index, 1)
        if (index <= this.active) {
          // 删除一个让索引减少1
          this.$emit('updata-active', this.active - 1, true)
        }
        // 单独封装一个方法处理数据持久化
        this.deleteChannel(channel)
      } else {
        // 如果是非编辑状态执行切换
        // 如果切换的话需要用到父组件的active状态
        // 所以需要获取子组件当前的索引传给父组件改变active
        this.$emit('updata-active', index, false)
      }
    },
    async deleteChannel (channel) {
      try {
        if (this.user) {
          // 已登录，则将数据更新到线上
          await deleteUserChannel(channel.id)
        } else {
          // 未登录，将数据更新到本地
          // 从新存一遍就相当于修改了
          setItem('TOUTIAO_CHANNELS', this.myChannelsList)
        }
      } catch (err) {
        this.$toast('失败')
      }
    }
  }
}
</script>

<style scoped lang="less">
.channel-edit {
  padding: 85px 0;

  .title-text {
    font-size: 32px;
    color: #333333;
  }
  .edit-btn {
    width: 104px;
    height: 48px;
    font-size: 26px;
    color: #f85959;
    border: 1px solid #f85959;
  }
  /deep/ .grid-item {
    width: 160px;
    height: 86px;
    .van-grid-item__content {
      white-space: nowrap;
      background-color: #f4f5f6;
      .van-grid-item__text,
      .text {
        color: #222;
        font-size: 28px;
        margin-top: 0;
      }
      .active {
        color: red;
      }
      .van-grid-item__icon-wrapper {
        position: unset;
      }
      // .van-icon {
      //   font-size: 24px;
      // }
    }
  }
  /deep/ .my-grid {
    .grid-item {
      .van-icon-clear {
        position: absolute;
        right: -10px;
        top: -10px;
        font-size: 30px;
        color: #cacaca;
        z-index: 2;
      }
    }
  }

  /deep/ .recommend-grid {
    .grid-item {
      .van-grid-item__content {
        flex-direction: row;
        .van-icon-plus {
          font-size: 28px;
          margin-right: 6px;
        }
      }
    }
  }
  .van-button--plain.van-button--danger {
    width: 90px;
  }
}
</style>
