<template>
  <div>
    <div class="login">
    </div>
    <!--登录中间块-->
    <div class="login-mid">
      <div class="login-mid-top">
        <div class="shadow-top-left"></div>
        <div class="shadow-top-right"></div>
      </div>
      <div class="login-mid-mid">

        <!--捕获人脸区域-->
        <div class="videoCamera-canvasCamera">
          <video
            id="videoCamera"
            :width="videoWidth"
            :height="videoHeight"
            autoplay
          ></video>
          <canvas
            style="display: none"
            id="canvasCamera"
            :width="videoWidth"
            :height="videoHeight"
          ></canvas>

          <!--人脸特效区域-->
          <div v-if="faceImgState" class="face-special-effects-2"></div>
          <div v-else class="face-special-effects"></div>

        </div>

        <!--按钮区域-->
        <div class="face-btn">
          <button @click="faceVef()">{{ faceImgState ? '正在识别中...' : '开始识别' }}</button>
        </div>

        <!--消息区域-->
        <div class="msg">
          <div class="server-msg">{{ msg }}</div>
        </div>
      </div>
      <div class="login-mid-bot">
        <div class="shadow-bot-left"></div>
        <div class="shadow-bot-right"></div>
      </div>
    </div>
    <login-select-tenant ref="loginSelect" @success="loginSelectOk"></login-select-tenant>
  </div>
</template>
<script>
import $camera from './face/index.js'
// import $login from '../src/store/modules/user.js'
import { timeFix, welcome } from '@/utils/util'
import LoginSelectTenant from '../LoginSelectTenant.vue'
import Vue from 'vue'
import { ACCESS_TOKEN, UI_CACHE_DB_DICT_DATA, USER_INFO, USER_NAME } from '@/store/mutation-types'

export default {
  components: { LoginSelectTenant },
  data() {
    return {
      videoWidth: 200,
      videoHeight: 200,
      msg: '',
      faceImgState: false,
      faceOption: {}
    }
  },
  mounted() {
    //调用摄像头
    this.faceOption = $camera.getCamera({
      videoWidth: 200,
      videoHeight: 200,
      thisCancas: null,
      thisContext: null,
      thisVideo: null,
      canvasId: 'canvasCamera',
      videoId: 'videoCamera'
    })
  },
  methods: {
    // 调用后台接口
    faceVef() {
      // 开始绘制图片
      let imageBase = $camera.draw(this.faceOption)
      if (this.faceImgState) {
        return
      }
      this.faceImgState = true
      if (imageBase === '' || imageBase === null || imageBase === undefined) {
        this.$message.error('图片数据为空')
      } else {
        this.$http.post('/sys/faceLogin', { imageBase }).then(res => {
            this.faceImgState = false
            // 关闭摄像头
            this.faceOption.thisVideo.srcObject.getTracks()[0].stop()

            const result = res.result
            const userInfo = result.userInfo
            Vue.ls.set(ACCESS_TOKEN, result.token, 7 * 24 * 60 * 60 * 1000)
            Vue.ls.set(USER_NAME, userInfo.username, 7 * 24 * 60 * 60 * 1000)
            Vue.ls.set(USER_INFO, userInfo, 7 * 24 * 60 * 60 * 1000)
            Vue.ls.set(UI_CACHE_DB_DICT_DATA, result.sysAllDictItems, 7 * 24 * 60 * 60 * 1000)
            this.$store.commit('SET_TOKEN', result.token)
            this.$store.commit('SET_INFO', userInfo)
            this.$store.commit('SET_NAME', {
              username: userInfo.username,
              realname: userInfo.realname,
              welcome: welcome()
            })
            this.$store.commit('SET_AVATAR', userInfo.avatar)

            // this.$router.push('/home')
            this.loginSuccess()
            // if (res.data.code === 201) {
            //   this.$message.success(res.data.msg)
            // }
          }, () => {
            this.faceImgState = false
          }
        )
      }
    },
    requestFailed(err) {
      this.$notification['error']({
        message: '错误',
        description: ((err.response || {}).data || {}).message || '请求出现错误，请稍后再试',
        duration: 4
      })
      this.registerBtn = false
    },
    loginSelectOk() {
      this.loginSuccess()
    },
    //登录成功
    loginSuccess() {
      this.$router.push({ path: '/dashboard/analysis' }).catch(() => {
      })
      this.$notification.success({
        message: '欢迎',
        description: `${timeFix()}，欢迎回来`
      })
    }
  }
}
</script>
<style>
@import "face/index.css";
</style>