<template>
  <scroll-view id="myScrollView" scroll-y="true" style="height: 100vh">
    <view :style="'height:' + statusBarHeight + 'px;'"></view>
    <view class="res_box">
      <view class="res_box2">
        <img
          src="https://lanhu.oss-cn-beijing.aliyuncs.com/SketchPnge1553c1d8d43568ee582b54f1deab731573c8804790b09e299c2e73eda1cfa68"
          alt=""
          @click="res_goback"
        />
        <view class="res_text"> AI测评结果 </view>
      </view>

      <view class="res_box3">
        <view class="res_heard">
          <img style="border-radius: 50%" :src="userInfo.avatar" alt="" />
          <view class="head_text">
            <view class="text_1">
              {{ userInfo.nickname }}
            </view>
            <view class="text_2"> ID:{{ userInfo.id }} </view>
          </view>
        </view>
        <view class="score_box">
          <view class="score_box1">
            <view class="score_text">
              {{ comprehensiveLevel }}
            </view>
            <img
              src="https://lanhu.oss-cn-beijing.aliyuncs.com/SketchPngfe293373ca510260bf25cf15905e96d33c640dc3b4be40d673f30a4860f69215"
              alt=""
            />
            <img
              src="https://lanhu.oss-cn-beijing.aliyuncs.com/SketchPngfe293373ca510260bf25cf15905e96d33c640dc3b4be40d673f30a4860f69215"
              alt=""
            />
          </view>
          <view class="scoring_box">
            <view
              class="xingxing1"
              v-for="(item, index) in score_List"
              :key="index"
            >
              <view class="xingxing">
                {{ item.title }}
              </view>
              <uni-rate :readonly="true" size="20" :value="item.score" />
            </view>
          </view>
        </view>
        <view class="ipt_res" style="display: flex">
          <canvas
            ref="wordMask"
            canvas-id="wordMask"
            id="wordMask"
            style="
              width: 128px;
              height: 128px;
              transform: scale(0.7);
              margin-left: -16px;
              margin-top: -23px;
            "
          ></canvas>
          <view style="width: 60%" class="pageDesc">{{ evaluationDesc }}</view>
        </view>

        <!-- <textarea type="text" class="ipt_res"
					placeholder="  重心写的很棒，注意整体形态，笔画再加长一些，加油！你距离成功仅仅一步之遥，多跟泽曦名师学习，相信可以写的更好！" /> -->
      </view>
    </view>
    <view>
      <canvas
        ref="myCanvas"
        canvas-id="myCanvas"
        id="myCanvas"
        :style="{ width: '100vw', height: canvasHeight + 'px' }"
        @tap="handleCanvasClick"
      >
      </canvas>
    </view>
  </scroll-view>
</template>

<script>
import { goStudy, fetchHomeworkAIReviewDetail } from '@/utils/api'

export default {
  data() {
    return {
      canvasRef: null, // canvas 的引用
      stu_List: [
        {
          name: '刘琪琦',
          id: '1008601',
        },
      ],
      score_List: [
        {
          title: '重心',
          score: 5,
        },
        {
          title: '位置',
          score: 4,
        },
        {
          title: '形态',
          score: 3,
        },
        {
          title: '笔画',
          score: 2,
        },
        {
          title: '大小',
          score: 1,
        },
      ],
      comprehensiveLevel: '',
      imagePath: '',
      token: null,
      pageMark: 0, // 卷面分
      canvasWidth: 0, // canvas 的宽度
      canvasHeight: 0, // canvas 的高度，根据图片比例动态设置
      chooseImageWidth: 0, // px
      chooseImageHeight: 0, // px
      ocrResults: null,
      wordMarks: null,
      scrollTop: 0,
      mouseX: 0,
      mouseY: 0,
      evaluationDesc: '',
      statusBarHeight: '',
      historyAi_id: '',
      originImg: null,
      userInfo: null,
    }
  },
  //   computed: {
  //     userInfo() {
  //       return uni.getStorageSync('info')
  //     },
  //   },
  mounted() {
    this.getToken() // 在组件挂载时获取 token
  },
  onLoad(options) {
    this.userInfo = uni.getStorageSync('info')
    this.historyAi_id = options.homework_id
    // 获取设备屏幕宽度，用于设置 canvas 宽度
    const systemInfo = uni.getSystemInfoSync()
    this.canvasWidth = systemInfo.windowWidth // 设置 canvas 宽度为 100vw
    if (!this.historyAi_id) {
      console.log(11)
      //   this.chooseImage()
    } else {
      this.getFetchHomeworkAIReviewDetail()
    }
    this.statusBarHeight = getApp().globalData.top
  },
  // methods: {
  //   handleCanvasClick() {
  //     console.log(125464646)
  //   }
  // },
  methods: {
    score() {
      let scoreText = this.evaluationDesc

      if (scoreText) {
        // 使用正则表达式提取分数
        const scores = {
          综合得分: null,
          重心得分: null,
          形态得分: null,
          大小得分: null,
          位置得分: null,
          笔画得分: null,
        }

        const regex =
          /(综合得分|重心得分|形态得分|大小得分|位置得分|笔画得分)[^\d]*(\d+)/g
        let match
        while ((match = regex.exec(scoreText)) !== null) {
          const scoreType = match[1] // 得分类型
          const scoreValue = parseInt(match[2], 10) // 将得分值转换为整数
          if (scores.hasOwnProperty(scoreType)) {
            scores[scoreType] = scoreValue // 存储得分
          }
        }

        const getComprehensiveLevel = (score) => {
          return score >= 60 ? '优' : '差' // 60分及以上为优
        }

        if (scores.综合得分 !== null) {
          this.comprehensiveLevel = getComprehensiveLevel(scores.综合得分)
        }

        const getOtherLevel = (score) => {
          if (score >= 80) return 5 // 80-100 -> 等级5
          if (score >= 60) return 4 // 60-79 -> 等级4
          if (score >= 40) return 3 // 40-59 -> 等级3
          if (score >= 20) return 2 // 20-39 -> 等级2
          return 1 // 0-19 -> 等级1
        }

        const scoreMap = {
          重心: scores.重心得分,
          位置: scores.位置得分,
          形态: scores.形态得分,
          笔画: scores.笔画得分,
          大小: scores.大小得分,
        }

        this.score_List.forEach((item) => {
          const score = scoreMap[item.title]
          if (score !== undefined) {
            item.score = getOtherLevel(score) // 将等级赋值给对应的项
          }
        })
      }
    },
    getToken() {
      uni.getStorage({
        key: 'token', // 存储的 key 名
        success: (res) => {
          this.token = res.data // 将获取的 token 存储到 data 属性中
          console.log('获取的 token:', this.token)
        },
        fail: (err) => {
          console.error('获取 token 失败:', err)
        },
      })
    },
    res_goback() {
      console.log(123)
      uni.navigateBack({
        delta: 1,
      })
    },

    async getPrivateDoc(targetFile) {
      return new Promise((resolve, reject) => {
        let fileReader = new plus.io.FileReader() //new一个可以用来读取文件的对象fileReader
        fileReader.readAsText(targetFile, 'utf-8') //读文件的格式
        fileReader.onerror = (e) => {
          console.log('获取文件失败', e)
          console.log('获取文件失败', fileReader.error)
          plus.nativeUI.toast('获取文件失败,请重启应用', {
            background: '#ffa38c',
          })
          return
        }
        fileReader.onload = (e) => {
          console.log('读取文件成功')
          let txtData = e.target.result
          resolve(txtData)
        }
      })
    },

    handleshadowImageLoad(e) {
      console.log('handleshadowImageLoad', e)
    },

    async getFetchHomeworkAIReviewDetail() {
      try {
        const data = {
          homework_id: this.historyAi_id,
        }
        const res = await fetchHomeworkAIReviewDetail(data)
        // console.log(res)

        this.pageMark = JSON.parse(res.items[0].content).pageMark
        this.ocrResults = JSON.parse(res.items[0].content).ocrResults
        this.wordMarks = JSON.parse(res.items[0].content).wordMarks
        const imagePath = res.items[0].origin_img // 使用可选链操作符避免未定义的错误
        const ctx = uni.createCanvasContext('myCanvas')
        // console.log(imagePath)

        uni.getImageInfo({
          src: imagePath,
          success: (image) => {
            // console.log('image width', image.width)
            // console.log('image height', image.height)

            this.chooseImageWidth = image.width
            this.chooseImageHeight = image.height

            const aspectRatio = this.chooseImageHeight / this.chooseImageWidth // 计算图片宽高比
            this.canvasHeight = this.canvasWidth * aspectRatio
            this.imagePath = imagePath
            // console.log(this.imagePath)
            this.drawImage()
            this.drawImageWithData()
          },
        })
      } catch (err) {}
    },

    chooseImage() {
      this.pageMark = 0
      uni.chooseImage({
        count: 1,
        sizeType: ['compressed'],
        sourceType: ['album', 'camera'],
        success: (res) => {
          this.imageSrc = res.tempFilePaths[0]
          this.imagePath = res.tempFilePaths[0]

          //   console.log('tempFilePaths-0 ', res.tempFilePaths[0])

          //   console.log(this.imagePath)
          //   console.log('choose file path ', res.tempFiles[0].path)
          //   console.log('choose file size ', res.tempFiles[0].size)

          uni.getImageInfo({
            src: res.tempFilePaths[0],
            success: (image) => {
              console.log('choose image width', image.width)
              console.log('choose image height', image.height)
              this.chooseImageWidth = image.width
              this.chooseImageHeight = image.height
              const aspectRatio = this.chooseImageHeight / this.chooseImageWidth // 计算图片宽高比
              this.canvasHeight = this.canvasWidth * aspectRatio
              setTimeout(() => {
                this.drawImage()
                this.submitForm()
              }, 100)
            },
          })
        },
        fail: (err) => {
          // console.error('选择图片失败：', err)
          uni.navigateBack() // 返回上一页
        },
      })
    },
    submitForm() {
      if (!this.imagePath) {
        return uni.showToast({
          title: '请先选择图片',
          icon: 'none',
        })
      }

      if (!this.token) {
        return uni.showToast({
          title: '请先登录',
          icon: 'none',
        })
      }

      uni.uploadFile({
        url: 'http://admin.zexishuhua.com/api/common/detecthomework',
        // url: 'http://zexiart.com/api/common/detecthomework',
        filePath: this.imagePath,
        name: 'file',
        header: {
          token: `${this.token}`,
        },
        formData: {
          sdk: 'WordEvaluation',
          device_id: 'test_device_id', // 随便填
          isCheckExamNum: 'false',
          font: 0,
          // "userId": user,
        },
        success: (uploadFileRes) => {
          const response = JSON.parse(uploadFileRes.data)
          if (response.code == 0) {
            this.pageMark = response.data.pageMark
            this.ocrResults = response.data.ocrResults
            this.wordMarks = response.data.wordMarks
            this.drawImageWithData()
            // this.drawImageWithData()
          }
        },
        fail: (err) => {
          // console.error('提交失败：', err)
          uni.showToast({
            title: '提交失败',
            icon: 'none',
          })
        },
      })
    },

    drawImage() {
      const ctx = uni.createCanvasContext('myCanvas')
      console.log(ctx, '555')
      ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)
      console.log('before ctx.drawimage')
      ctx.drawImage(this.imagePath, 0, 0, this.canvasWidth, this.canvasHeight) // 按比例绘制图片
      console.log('after ctx.drawimage')
      ctx.draw(false, () => {
        console.log('draw image success')
      })
    },

    handleCanvasClick(event) {
      const query = uni.createSelectorQuery().in(this)
      query
        .select('#myScrollView')
        .scrollOffset((data) => {
          const x = event.detail.x - event.target.offsetLeft
          const y = event.detail.y - event.target.offsetTop
          this.mouseX = x
          this.mouseY = y + data.scrollTop
          this.drawImageWithData()
        })
        .exec()
    },
    drawImageWithData() {
      const ctx = uni.createCanvasContext('myCanvas')
      ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)
      ctx.drawImage(this.imagePath, 0, 0, this.canvasWidth, this.canvasHeight) // 按比例绘制图片

      if (!this.ocrResults && !this.wordMarks) {
        return
      }

      const aspectRatio = this.chooseImageHeight / this.chooseImageWidth // 计算图片宽高比
      const scale = this.canvasWidth / this.chooseImageWidth

      this.ocrResults.forEach((ocr, i) => {
        const word = this.wordMarks[i]
        let x = ocr.points[0] * scale
        let y = ocr.points[1] * scale
        let width = (ocr.points[4] - ocr.points[0]) * scale
        let height = (ocr.points[5] - ocr.points[1]) * scale

        // 创建一个变量来存储显示的文本
        let displayText

        if (word.mark > 80) {
          displayText = '优' // 如果分数大于80，设置显示文本为"优"
        } else if (word.mark >= 70) {
          displayText = '良' // 如果分数在60到80之间，设置显示文本为"良"
        } else {
          displayText = '学习' // 如果分数低于60，设置显示文本为"去学习"
        }

        if (
          this.mouseX >= x &&
          this.mouseX <= x + width &&
          this.mouseY >= y &&
          this.mouseY <= y + height
        ) {
          ctx.strokeStyle = 'blue'
          ctx.fillStyle = 'blue'
          ctx.strokeRect(x, y, width, height)
          // ctx.fillText(`${word.mark.toFixed(0)}`, x, y + 25);
          ctx.fillText(displayText, x, y + 25)

          this.drawMask('wordMask', ocr.label, word.base64Mark)
          this.showWordsInfo(ocr, word)
          // 将点击的文字转化为 Base64 格式
          // 将点击的文字转化为 Base64 格式
          const base64Label = btoa(unescape(encodeURIComponent(ocr.label)))
          //   const encoder = new TextEncoder()
          //   const encodedData = encoder.encode(ocr.label) // 将文本转换为 Uint8Array
          //   const base64Label = btoa(String.fromCharCode(...encodedData)) // 转换为 Base64
          console.log(
            `你点击了文字 "${ocr.label}"，分数: ${word.mark}，Base64格式: ${base64Label}`
          )
          if (displayText === '学习') {
            let params = {
              fonts: base64Label,
            }
            goStudy(params)
              .then((res) => {
                console.log(res) // 打印响应
                if (res.code === 1) {
                  uni.navigateTo({
                    url: '/pages/pagesall/course/go_learn', // 替换为实际的页面路径
                  })
                } else {
                  uni.showToast({
                    title: res.msg,
                    icon: 'error',
                  })
                }
              })
              .catch((error) => {
                console.error('请求发生错误:', error) // 打印错误信息
                uni.showToast({
                  title: '请求失败，请稍后重试。',
                  icon: 'error',
                })
              })
          }
        } else {
          // 低分用不同颜色
          if (word.mark < 70) {
            ctx.strokeStyle = 'red'
            ctx.fillStyle = 'red'
          } else {
            ctx.strokeStyle = '#2e8b57'
            ctx.fillStyle = '#2e8b57'
          }
          ctx.strokeRect(x, y, width, height)
          ctx.fillText(displayText, x, y + 25)

          // ctx.fillText(`${word.mark.toFixed(0)}`, x, y + 25);
        }
      })

      if (this.mouseX == 0 && this.mouseY == 0 && this.ocrResults.length > 0) {
        this.drawMask(
          'wordMask',
          this.ocrResults[0].label,
          this.wordMarks[0].base64Mark
        )
        this.showWordsInfo(this.ocrResults[0], this.wordMarks[0])
      }

      ctx.draw(false, () => {
        console.log('draw image success')
      })
    },
    drawMask(canvasId, label, base64String) {
      const ctx = uni.createCanvasContext('wordMask', this)

      ctx.clearRect(0, 0, 128, 128)
      ctx.setFillStyle('#ffffff')
      ctx.fillRect(0, 0, 128, 128)

      // 标准字
      const x = 128 / 2
      const y = 128 / 2
      ctx.setFillStyle('#FF5733')
      ctx.font = '106px simkai'
      ctx.setFontSize(106)

      ctx.setTextAlign('center')
      ctx.setTextBaseline('middle')
      ctx.fillText(label, x, y)

      const binaryData = atob(base64String)
      const byteArray = new Uint8Array(binaryData.length)
      for (var i = 0; i < binaryData.length; i++) {
        byteArray[i] = binaryData.charCodeAt(i)
      }

      // Draw the image from the binary data
      function draw() {
        let bits = displayBits(byteArray)
        drawBinaryImage(bits)

        function drawBinaryImage(bits) {
          for (let i = 0; i < 128; ++i) {
            for (let j = 0; j < 128; ++j) {
              let bit = bits[128 * i + j]
              if (bit === 0) {
                const colorValue = bit === 0 ? 0 : 255
                ctx.setFillStyle(
                  `rgb(${colorValue}, ${colorValue}, ${colorValue})`
                )
                ctx.fillRect(j, i, 1, 1)
              }
            }
          }
        }

        function displayBits(uint8Array) {
          const bitValues = []
          for (let i = 0; i < uint8Array.length; i++) {
            for (let j = 7; j >= 0; j--) {
              const bit = (uint8Array[i] >> j) & 1
              bitValues.push(bit)
            }
          }
          return bitValues
        }
      }
      draw()

      ctx.draw(false, () => {
        console.log('draw mask success')
        // const store = useStore()
        // const canvasElement = this.$refs.aiBox // 获取 canvas 元素
      })
    },

    showWordsInfo(ocrResult, wordMark) {
      // 构建文本信息
      let sb = []
      sb.push(ocrResult.label + '\n')
      if (wordMark.mark <= 0) {
        sb.push('暂无识别字的评测数据\n')
      }
      sb.push('综合得分 ' + wordMark.mark + '\n')
      sb.push('重心得分 ' + wordMark.centerMark)

      if (wordMark.centerInfo === 0) {
        sb.push('良好')
      } else if (wordMark.centerInfo === 1) {
        sb.push('偏上')
      } else if (wordMark.centerInfo === 2) {
        sb.push('偏下')
      } else if (wordMark.centerInfo === 3) {
        sb.push('偏左')
      } else if (wordMark.centerInfo === 4) {
        sb.push('偏右')
      }
      sb.push('\n')

      sb.push('形态得分 ' + wordMark.rateMark)
      if (wordMark.rateInfo === 0) {
        sb.push('良好')
      } else if (wordMark.rateInfo === 1) {
        sb.push('太宽')
      } else if (wordMark.rateInfo === 2) {
        sb.push('太瘦')
      }
      sb.push('\n')

      sb.push('大小得分 ' + wordMark.areaMark)
      if (wordMark.areaInfo === 0) {
        sb.push('良好')
      } else if (wordMark.areaInfo === 1) {
        sb.push('太小')
      } else if (wordMark.areaInfo === 2) {
        sb.push('太大')
      }
      sb.push('\n')

      sb.push('位置得分 ' + wordMark.posMark)
      if (wordMark.posInfo === 0) {
        sb.push('良好')
      } else if (wordMark.posInfo === 1) {
        sb.push('偏上')
      } else if (wordMark.posInfo === 2) {
        sb.push('偏下')
      } else if (wordMark.posInfo === 3) {
        sb.push('偏左')
      } else if (wordMark.posInfo === 4) {
        sb.push('偏右')
      }
      sb.push('\n')

      sb.push('笔画得分 ' + wordMark.conFMark)
      if (wordMark.conFInfo === 0) {
        sb.push('良好')
      } else if (wordMark.conFInfo === 1) {
        sb.push('不良')
      }
      sb.push('\n')

      this.evaluationDesc = sb.join('')
      this.score()
      // console.log(this.evaluationDesc)
      // const store = useStore()
      // store.setEvaluationDesc(this.evaluationDesc)
    },
  },
}
</script>

<style>
.res_box {
  width: 750rpx;
  height: 850rpx;
  background-color: rgba(249, 183, 34, 1);
}

.res_box2 {
  position: relative;
  padding: 30rpx;
}

.res_box2 img {
  position: absolute;
  top: 40rpx;
}

.res_text {
  text-align: center;
  font-weight: 500;
  font-size: 36rpx;
  color: #ffffff;
}

.res_box3 {
  width: 638rpx;
  height: 632rpx;
  background: #ffffff;
  border-radius: 16rpx;
  margin-left: 28rpx;
  padding: 28rpx;
  margin-top: 30rpx;
}

.res_heard {
  display: flex;
  align-items: center;
}

.res_heard img {
  width: 144rpx;
  height: 144rpx;
}

.head_text {
  margin-left: 28rpx;
}

.text_2 {
  margin-top: 8rpx;
  font-weight: 400;
  font-size: 24rpx;
  color: #000000;
}

.text_1 {
  font-weight: 500;
  font-size: 36rpx;
  color: #000000;
}

.score_text {
  width: 128rpx;
  height: 180rpx;
  overflow-wrap: break-word;
  color: rgba(219, 1, 12, 1);
  font-size: 128rpx;
  font-family: PingFangSC-Medium;
  font-weight: 500;
  text-align: left;
  white-space: nowrap;
  line-height: 180rpx;
}

.score_box1 {
  display: flex;
  align-items: center;
  flex-direction: column;
  justify-content: center;
}

.score_box {
  display: flex;
  justify-content: space-between;
}

.xingxing1 {
  width: 284rpx;
  height: 48rpx;
  display: flex;
  align-items: center;
  justify-content: space-around;
  padding: 5rpx 0;
}

.xingxing {
  font-weight: 600;
  font-size: 28rpx;
  color: #000000;
  line-height: 48rpx;
  text-align: left;
}

.ipt_res {
  width: 608rpx;
  height: 164rpx;
  background: #f1f1f1;
  border-radius: 8rpx;
  padding: 15rpx;
  margin-top: 15rpx;
  font-size: 28rpx;
}

.score_img {
  width: 870rpx;
  height: 946rpx;
}

.score_line {
  width: 870rpx;
  height: 1000rpx;
  background-color: rgba(249, 183, 34, 1);
}
</style>
