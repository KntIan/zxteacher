<template>
  <view>
    <view :style="'height:' + (statusBarHeight + 5) + 'px;'"></view>
    <view class="chat_box" :scroll-into-view="poaMessgae">
      <view class="chat_title_box">
        <view class="chat_title">
          <view class="chat_title_img" @click="chat_goback">
            <img
              src="https://lanhu.oss-cn-beijing.aliyuncs.com/SketchPng427bd6433cc6e0a8e82f63b3174b2c817dc9c299bd0c4414c8d258f46cf46f94"
              alt=""
            />
          </view>
          <view class="chat_head_img">
            <img
              src="https://lanhu-dds-backend.oss-cn-beijing.aliyuncs.com/merge_image/imgs/cf805a0bbe4b4f17abb1c215c9ec2df7_mergeImage.png"
              alt=""
            />
          </view>
          <view class="chat_name">
            {{ chat_name }}
          </view>
        </view>
      </view>

      <view class="chat-container">
        <scroll-view
          ref="messageList"
          scroll-y="true"
          class="message-list"
          :scroll-into-view="poaMessgae"
        >
          <view
            v-for="(item, index) in messages_stu"
            :key="index"
            class="message-item_stu"
          >
            <view
              class="time"
              v-if="
                index == 0 || item.time - messages_stu[index - 1].time >= 300000
              "
            >
              <view>{{ formatDate(item.time) }}</view>
            </view>
            <view class="content1" :id="item.status == 'l' ? 'l' : 'r'">
              <view class="text" v-if="item.type == 'img'">
                <img
                  src="https://lanhu.oss-cn-beijing.aliyuncs.com/SketchPng580a34d876cda0a5b37d451367934eb478865bb7b4b80bedde3f71fcaa5c8775"
                  alt=""
                />
              </view>
              <view class="advio" v-if="item.type == 'record'">
                <view class=""> {{ item.text }} " </view>
                <img src="@/static/img/advio.png" alt="" />
              </view>
              <view class="text" v-if="item.type == 'text'">{{
                item.text
              }}</view>
              <view class="avatar" style="margin-left: 20rpx">
                <image :src="item.avatar"></image>
              </view>
            </view>
          </view>
          <view id="poaMessgae"></view>
          <!-- 仅用于定位到消息最后一条 -->
        </scroll-view>
        <view class="input-container">
          <view class="chat_input">
            <img
              src="https://lanhu.oss-cn-beijing.aliyuncs.com/SketchPng8336930e00a2e304b3bd63727ff79ab21eb7c0da57bcc1da2dc88adc00ef2cbe"
              alt=""
              @click="toggleInputMode"
              v-if="change"
            />
            <img
              src="@/static/img/Frame.png"
              alt=""
              srcset=""
              @click="toggleInputMode"
              v-else
            />
            <input
              v-model="inputMessage"
              placeholder="发消息..."
              @focus="scrollToBottom"
              confirm-type="send"
              @confirm="onSend"
              v-if="change"
            />

            <view
              class="voice"
              @touchstart="handleTouchStart"
              @touchend="handleTouchEnd"
              v-else
            >
              按住说话
            </view>
            <img
              src="https://lanhu.oss-cn-beijing.aliyuncs.com/SketchPng59f0b59c28758e28c9970d6e3110b53be29f04b99fded460a8f2a1cd5b32d1da"
              alt=""
              class="btn_img"
            />
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      statusBarHeight: '',
      messages_stu: [
        {
          username: '刘易阳',
          type: 'text',
          status: 'l',
          avatar:
            'https://lanhu-dds-backend.oss-cn-beijing.aliyuncs.com/merge_image/imgs/cf805a0bbe4b4f17abb1c215c9ec2df7_mergeImage.png',
          text: '刘易阳你今天表现不错哦1111111111111111111111111111111111111111111111',
          time: 1683791638972,
        },
        // 其他学生消息...
      ],
      inputMessage: '',
      chat_name: '书法一班刘老师',
      change: true,
      poaMessgae: 'poaMessgae',
      scrollViewHeight: '',
      startTime: null,
      longPressDuration: 6000, // 定义长按的阈值（毫秒）,
      show: true,
    }
  },
  methods: {
    onload() {
      console.log('1111')
      this.scrollToBottom()
      this.poalast() // 定位消息最后一行
      this.getHeight('.chat_box')
      this.statusBarHeight = getApp().globalData.top
    },
    formatDate(value) {
      if (typeof value == 'undefined') return ''

      const date = new Date(value)
      const now = new Date()
      const y = date.getFullYear()
      const MM = String(date.getMonth() + 1).padStart(2, '0')
      const d = String(date.getDate()).padStart(2, '0')
      const h = String(date.getHours()).padStart(2, '0')
      const m = String(date.getMinutes()).padStart(2, '0')

      if (now.getDate() - d == 1 && now - date < 172800000) {
        return `昨天 ${h}:${m}`
      } else if (now - date < 86400000) {
        return `${h}:${m}`
      } else if (now - date >= 86400000 && now - date < 31536000000) {
        return `${MM}-${d} ${h}:${m}`
      } else if (now - date >= 31536000000) {
        return `${y}-${MM}-${d} ${h}:${m}`
      }
    },
    onSend() {
      const mess = {
        text: this.inputMessage,
        status: 'r',
        type: 'text',
        avatar:
          'https://lanhu-dds-backend.oss-cn-beijing.aliyuncs.com/merge_image/imgs/7872400aa4644186b920b0121d6a5136_mergeImage.png',
        time: new Date() - 0,
      }
      this.sendmessage(JSON.stringify(mess))
      this.inputMessage = ''
    },
    sendmessage(e) {
      try {
        console.log(e)
        this.messages_stu.push(JSON.parse(e))
        this.scrollToBottom()
        this.poalast() // 定位消息最后一行
      } catch (error) {
        console.error('消息推送失败:', error)
      }
    },
    scrollToBottom() {
      this.$nextTick(() => {
        const scrollView = this.$refs.messageList
        if (scrollView) {
          scrollView.scrollTop = scrollView.scrollHeight
        }
      })
    },
    poalast() {
      this.$nextTick(() => {
        this.poaMessgae = ''
        setTimeout(() => {
          this.poaMessgae = 'poaMessgae'
        }, 50)
      })
    },
    toggleInputMode() {
      this.change = !this.change
    },
    handleTouchStart() {
      this.startTime = new Date().getTime()
    },
    handleTouchEnd() {
      if (this.startTime) {
        const endTime = new Date().getTime()
        const duration = Math.round((endTime - this.startTime) / 1000)
        if (duration < 1) {
          console.log('时间太短了')
        } else {
          const mess = {
            username: '刘易阳',
            type: 'record',
            status: 'r',
            avatar:
              'https://lanhu-dds-backend.oss-cn-beijing.aliyuncs.com/merge_image/imgs/7872400aa4644186b920b0121d6a5136_mergeImage.png',
            text: duration,
            time: new Date() - 0,
          }
          this.messages_stu.push(mess)
          this.poalast() // 定位消息最后一行
        }
        this.startTime = null // 重置 startTime
      }
    },
    chat_goback() {
      uni.navigateBack()
    },
  },
  mounted() {
    this.poalast() // 定位消息最后一行
  },
}
</script>

<style scoped>
.chat-container {
  display: flex;
  flex-direction: column;
  height: 80vh;
  position: relative;
  padding-top: 120rpx;
  padding-bottom: 200rpx;
}

.chat_title_box {
  position: fixed;
  top: 0;
  width: 100%;
  background-color: #eee;
  z-index: 1000;
  height: 120rpx;
}

.message-list {
  flex: 1;
  background-color: #eee;
  overflow-y: scroll;
  width: 100%;
  height: 1500rpx;
}

.message-item {
  display: flex;
  margin-bottom: 10px;
}

.avatar {
  width: 40px;
  height: 40px;
  margin-right: 10px;
}

.avatar image {
  width: 100%;
  height: 100%;
  border-radius: 50%;
}

.content {
  flex: 1;
}

.username {
  font-weight: bold;
}

.text {
  margin-top: 5px;
}

.input-container {
  display: flex;
  padding: 30rpx 20rpx 20rpx 20rpx;
  background-color: #fff;
  align-items: flex-start;
  position: fixed;
  bottom: 0;
  width: 95%;
  z-index: 1000;
  height: 140rpx;
  border-radius: 30rpx 30rpx 0rpx 0rpx;
}

input {
  flex: 1;
  border: none;
  padding: 10px;
  font-size: 16px;
  border-radius: 20px;
  margin-right: 10px;
  margin-left: 10px;
  background-color: #eee;
}

.chat_title {
  display: flex;
  align-items: center;
  background-color: #eee;
}

.chat_title view {
  padding: 15rpx;
}

.chat_name {
  font-weight: 600;
  font-size: 36rpx;
  color: #000000;
}

.text {
  max-width: 460rpx;
  background-color: #fff;
  padding: 20rpx;
  border-radius: 20rpx;
  word-wrap: break-word;
  word-break: break-all;
  white-space: normal;
}

.message-item_stu {
  display: flex;
  justify-content: end;
  margin-top: 10rpx;
  margin-right: 10rpx;
  flex-direction: column;
}

.chat_box {
  background: #eeeeee;
}

.chat_input {
  display: flex;
  align-items: center;
}

.chat_input input {
  width: 558rpx;
}

.chat_input img {
  width: 40rpx;
  height: 40rpx;
}

.time {
  text-align: center;
  font-weight: 400;
  font-size: 20rpx;
  color: #000000;
  margin: 20rpx 0;
}

.content1 {
  display: flex;
  align-items: center;
  justify-content: end;
}

#l {
  flex-direction: row-reverse;
  justify-content: flex-end;
}

.voice {
  width: 558rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20rpx 25rpx;
  background: #eeeeee;
  border-radius: 38rpx;
  margin-left: 10rpx;
  margin-right: 10rpx;
}

.advio {
  width: 150rpx;
  background-color: #fff;
  padding: 10rpx;
  border-radius: 20rpx;
  display: flex;
  align-items: center;
  justify-content: end;
}

.advio img {
  width: 40rpx;
  height: 40rpx;
  margin-left: 30rpx;
  transform: scaleX(-1) scaleY(-1);
}

.box_show {
  background-color: red;
  width: 750rpx;
  height: 1624rpx;
  position: fixed;
  top: 0;
  left: 0;
}
</style>
