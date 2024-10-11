<template>
  <div>
    <div ref="anim" class="anim-container"></div>
    <button :class="[!access && 'disable']" @click.stop="play(0)">play(无融合)</button>
    <button :class="[!access && 'disable']" @click.stop="play(1)">play(有融合)</button>
    <button v-if="vap" @click.stop="playContinue()">continue</button>
    <button v-if="vap" @click.stop="pause()">pause</button>
  </div>
</template>

<script>
import Vap from '../../../dist/vap.js'
var TAG = 'vue端log：'

export default {
  name: 'vap',
  props: {
    demoConfig: Object,
    demo2Config: Object,
    demoMp4: String,
    demo2Mp4: String
  },
  data () {
    return {
      vap: null,
      access: true,
      url: this.demoMp4,
      config: this.demoConfig,
      fusionInfo: {
        // 融合信息（图片/文字）,同素材生成工具生成的配置文件中的srcTag所对应，比如[imgUser] => imgUser
        imgUser: '//shp.qlogo.cn/pghead/Q3auHgzwzM6TmnCKHzBcyxVPEJ5t4Ria7H18tYJyM40c/0',
        imgAnchor: '//shp.qlogo.cn/pghead/PiajxSqBRaEKRa1v87G8wh37GibiaosmfU334GBWgk7aC8/140',
        textUser: 'demo1 textUser',
        textAnchor: 'demo1 textAnchor',
        type: 2
      }
    }
  },
  created () {
    window.sendData = this.sendData.bind(this)
    this.vap = new Vap()
  },
  methods: {
    sendData (data) {
      console.log(TAG, '接收flutter参数：', data)
      try {
        const parsedData = JSON.parse(data) // 解析接收到的JSON字符串
        this.config = parsedData.demo1Json
        this.demo2Config = parsedData.demo2Json
        this.url = parsedData.demo1Mp4
        this.demo2Mp4 = parsedData.demo2Mp4
      } catch (error) {
        console.error(TAG, '解析错误：', error)
      }
    },
    getPlayParams () {
      return {
        container: this.$refs.anim,
        // 素材视频链接
        src: this.url,
        // 素材配置json对象
        config: this.config,
        width: 900,
        height: 400,
        // 同素材生成工具中配置的保持一致
        fps: 20,
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
      // 确保参数已正确初始化
      if (!this.url || !this.config || !this.$refs.anim) {
        console.error(TAG, '参数出错：src(视频地址)、config(配置文件地址)、container(dom容器)')
        return
      }
      const that = this
      const params = this.getPlayParams()
      // 检查 vap 实例状态
      if (!this.vap) {
        this.vap = new Vap()
      }
      this.vap.play(Object.assign({}, params, flag ? this.fusionInfo : {type: 1}))
        .on('playing', () => {
          that.access = false
          console.log('playing')
        })
        .on('ended', () => {
          that.access = true
          // this.vap = null
          console.log('play ended')
          if (type !== 'demo2') {
            this.changeConfig(this.demo2Mp4, this.demo2Config, 'demo2 textUser', 'demo2 textAnchor')
            this.play(flag, 'demo2')
          } else {
            this.changeConfig(this.demoMp4, this.demoConfig, 'demo1 textUser', 'demo1 textAnchor')
          }
        })
        .on('frame', (frame, timestamp) => {
          // frame: 当前帧(从0开始)  timestamp: (播放时间戳)
        })
      window.vap = this.vap
    },
    pause () {
      this.vap.pause()
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
.anim-container {
  width: 900px;
  height: 600px;
  border: 1px solid #cccccc;
  margin: auto;
  margin-bottom: 20px;
}
  button.disable {
    background: gray;
  }
</style>
