<template>
  <div>
    <div>
      <div>
        <video-component v-if="showVideo" id="myVideoEl" ref="myVideoEl" :my-stream-src="srcObject" width="320" />
      </div>
      <el-form ref="form" label-width="80px">
        <el-form-item>
          <el-button v-if="!showVideo" @click="startPublish">开始推流</el-button>
          <el-button v-if="showVideo" @click="stopPublish">结束推流</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div>SessionID: <span v-text="sessionid" /></div>
    <div>Audio: <span v-text="acodecs" /></div>
    <div>Video: <span v-text="vcodecs" /></div>
    <div>Simulator: <a :href="simulator">Drop</a></div>
  </div>
</template>

<script>
import VideoComponent from '../videoComponent'
import { SrsRtcPublisherAsync, SrsRtcFormatSenders } from '@/utils/srs.sdk'
export default {
  components: {
    VideoComponent
  },
  data() {
    return {
      showVideo: false,
      acodecs: '',
      vcodecs: '',
      sdk: '',
      sessionid: '',
      srcObject: '',
      url: 'webrtc://192.168.1.108/live/show',
      simulator: ''
    }
  },
  watch: {
    srcObject: function(newValue, oldValue) {
      const _this = this
      if (!oldValue && !!newValue) {
        _this.$nextTick(function() {
          setTimeout(() => {
            const myVideoEl = document.querySelector('#myVideoEl')
            myVideoEl.srcObject = _this.srcObject
          })
        })
      }
    }
  },
  methods: {
    startPublish() {
      const _this = this
      this.sdk = null // Global handler to do cleanup when republishing.

      if (this.sdk) {
        this.sdk.close()
      }
      this.sdk = new SrsRtcPublisherAsync()
      setTimeout(() => {
        _this.$nextTick(() => {
          _this.srcObject = this.sdk.stream
        })
        this.showVideo = true
        this.showVideo = false
        this.showVideo = true
      }, 100)

      this.sdk.pc.onicegatheringstatechange = function(event) {
        if (_this.sdk.pc.iceGatheringState === 'complete') {
          _this.acodecs = SrsRtcFormatSenders(_this.sdk.pc.getSenders(), 'audio')
          _this.vcodecs = SrsRtcFormatSenders(_this.sdk.pc.getSenders(), 'video')
        }
      }
      this.sdk.publish(this.url).then(function(session) {
        _this.sessionid = session.sessionid
        _this.simulator = session.simulator + '?drop=1&username=' + session.sessionid
      }).catch(function(reason) {
        _this.sdk.close()
        _this.showVideo = false
        console.error(reason)
      })
    },
    stopPublish() {
      if (this.showVideo) {
        if (document.getElementById('myVideoEl').srcObject.getTracks().length > 0) {
          const medias = document.getElementById('myVideoEl').srcObject.getTracks()
          for (let i = 0; i < medias.length; i++) {
            medias[i].stop()
          }
        }
        this.showVideo = false
      }
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
