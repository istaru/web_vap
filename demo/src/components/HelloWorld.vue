<template>
  <div>
    <div ref="anim" class="anim-container"></div>
    <button :class="[!access && 'disable']" @click.stop="play(0)">播放</button>
    <!-- <button :class="[!access && 'disable']" @click.stop="play(1)">play(有融合)</button> -->
    <button v-if="vap" @click.stop="pause()">暂停/继续</button>
    <!-- <button v-if="vap" @click.stop="playContinue()">resume</button> -->
  </div>
</template>

<script>
import Vap from '../../../dist/vap.js'
// import demoConfig from './demo1.json'
// import demo2Config from './demo2.json'
// const demoMp4 = require('./demo1.mp4')
// const demo2Mp4 = require('./demo2.mp4')
var TAG = 'vue端log：'

export default {
  name: 'vap',
  data () {
    return {
      vap: null,
      access: true,
      isOnPause: false,
      url: '',
      config: {},
      fusionInfo: {}
    }
  },
  created () {
    this.vap = new Vap()
  },
  mounted () {
    window.handleFlutterData = this.handleFlutterData
    // 网页加载完毕时，自动调用 play 方法
    this.play(0) // 传入参数 0 表示无融合播放
  },
  methods: {
    handleFlutterData (event) {
      console.log(TAG, '接收的参数为：', event)
      const parsedData = JSON.parse(event) // 解析接收到的JSON字符串
      // 更新状态
      this.url = parsedData.demo1Mp4 // 设置视频链接
      this.config = parsedData.demo1Json // 设置配置文件
    },
    getPlayParams () {
      return {
        container: this.$refs.anim,
        // 素材视频链接
        src: this.url,
        // 素材配置json对象
        config: this.config,
        width: 924 / 1.5,
        height: 492 / 1.5,
        // 同素材生成工具中配置的保持一致
        fps: 30,
        // 是否循环
        loop: false,
        // 起始播放时间点
        beginPoint: 0,
        // 精准模式
        accurate: true
        // 播放起始时间点(秒)
      }
    },
    play (flag, type = '') {
      if (!this.access) {
        return
      }
      const that = this
      const params = this.getPlayParams()
      // 记录开始播放时间
      const startTime = performance.now()
      this.vap.play(Object.assign({}, params, flag ? this.fusionInfo : {type: 1}))
        .on('playing', () => {
          that.access = false
          // console.log('playing')
        })
        .on('ended', () => {
          that.access = true
          // this.vap = null
          // console.log('play ended')
          // 记录结束播放时间
          const endTime = performance.now()
          const duration = endTime - startTime
          console.log(TAG, '动画耗时：', duration, '毫秒')
          // if (type !== 'demo2') {
          //   this.changeConfig(demo2Mp4, demo2Config, 'demo2 textUser', 'demo2 textAnchor')
          //   this.play(flag, 'demo2')
          // } else {
          //   this.changeConfig(demoMp4, demoConfig, 'demo1 textUser', 'demo1 textAnchor')
          // }
        })
        .on('frame', (frame, timestamp) => {
          // frame: 当前帧(从0开始)  timestamp: (播放时间戳)
          if (frame === 200) {
            // do something
          }
          // console.log(frame, '-------', timestamp)
        })
      window.vap = this.vap
    },
    pause () {
      if (this.access) {
        return
      }
      if (this.isOnPause) {
        this.isOnPause = false
        this.vap.play()
      } else {
        this.isOnPause = true
        this.vap.pause()
      }
    },
    playContinue () {
      this.vap.play()
    },
    changeConfig (demo2Mp4, demo2Config, textUser, textAnchor) {
      this.url = demo2Mp4
      this.config = demo2Config
      this.fusionInfo = {
        // 融合信息（图片/文字）,同素材生成工具生成的配置文件中的srcTag所对应，比如[imgUser] => imgUser
        imgUser: '//shp.qlogo.cn/pghead/Q3auHgzwzM6TmnCKHzBcyxVPEJ5t4Ria7H18tYJyM40c/0',
        imgAnchor: '//shp.qlogo.cn/pghead/PiajxSqBRaEKRa1v87G8wh37GibiaosmfU334GBWgk7aC8/140',
        textUser: textUser,
        textAnchor: textAnchor,
        type: 2
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
html, body {
  overflow-x: hidden; /* 禁止横向滚动 */
  margin: 0;
  padding: 0;
  width: 100vw; /* 保证页面宽度与视口一致 */
}
.anim-container {
  width: 100%;
  max-width: 100vw; /* 限制容器的最大宽度 */
  overflow: hidden;  /* 禁止内容超出容器时触发滚动 */
  height: 264px;
  border: 1px solid #00000000;
  background-color: #FFFFFF;
  margin: auto;
  margin-bottom: 60px;
}
button.disable {
  height: 100%;
  background: gray;
  display: block;
  margin: 0 auto;
}
</style>
