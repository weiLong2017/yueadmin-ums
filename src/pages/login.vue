<template>
  <transition name="slide-fade">
    <div class="container">
      <div v-title :data-title="this.$route.name"></div>
      <el-form :model="loginForm" :rules="loginRules" ref="loginForm" class="login-form" :class="{'animated shake': invalid}">
        <h2 class="page-header">欢迎登录</h2>
        <el-form-item label="用户名" prop="username" :label-width="labelWith">
          <el-input v-model.string="loginForm.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密 码" prop="password" :label-width="labelWith">
          <el-input type="password" v-model="loginForm.password" placeholder="请输入密码"></el-input>
        </el-form-item>
        <el-form-item label="验证码" prop="authcode" :label-width="labelWith">
          <el-input type="text" v-model="loginForm.authcode" placeholder="请输入验证码" style="float: left; width: 65%; margin-right: 15px;"></el-input>
          <canvas id="canvasCode" width="80px" height="35px" class="canvas-code" @click="drawCode"></canvas>
        </el-form-item>
        <el-form-item label="登录角色" :label-width="labelWith" class="m-b-5">
          <el-select v-model="loginForm.isAdmin" class="el-select--block" placeholder="请选择角色">
            <el-option
              v-for="(item, index) in loginRoles"
              :label="item.label"
              :value="item.value"
              :key="index">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="记住密码" :label-width="labelWith" class="m-b-5">
          <el-checkbox-group v-model="loginForm.remember">
            <el-checkbox name="type"></el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        <el-form-item class="m-b-5">
          <el-button type="primary" class="el-button--block" :loading="logging" @click="submitForm">
            提 交
          </el-button>
        </el-form-item>
        <el-form-item style="display: none; margin: 0; text-align: center;">
          <router-link to="/register">尚未注册？</router-link>
          <span style="margin: 0 10px">|</span>
          <router-link to="/forgetPass">忘记密码</router-link>
        </el-form-item>
      </el-form>    
    </div>
  </transition>
</template>
<script>
import { requestLogin } from '../api'
import utils from '../assets/js/utils'
export default {
  data() {
    var validateCode = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入验证码'))
      } else if (value.toUpperCase() !== this.authCode.toUpperCase()) {
        callback(new Error('验证码错误'))
      } else {
        callback()
      }
    }
    return {
      labelWith: '90px',
      invalid: false,
      logging: false,
      authCode: '',
      loginForm: {
        username: '',
        password: '',
        authcode: '',
        remember: false,
        isAdmin: '1'
      },
      loginRoles: [{
        value: '0',
        label: '商户'
      }, {
        value: '1',
        label: '管理员'
      }],
      loginRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 5, message: '长度不少于5个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 8, max: 20, message: '长度在 8 到 20 个字符', trigger: 'blur'}
        ],
        authcode: [
          { required: true, validator: validateCode, trigger: 'blur'}
        ]
      }
    };
  },
  methods: {
    submitForm () {
      let self = this;
      this.$refs.loginForm.validate((valid) => {
        console.log(this.loginForm)
        if (valid && !this.logging) {
          this.logging = true
          requestLogin(this.loginForm).then(res => {
            console.log(res)
            if (res.data.code === 0) {
              let result = res.data.result;
              console.log(result)
              localStorage.setItem('sessionId', result.session)
              localStorage.setItem('user', JSON.stringify(result.user))
              console.log(localStorage.getItem('user'))
              if (this.loginForm.remember) {
                let name = btoa(escape(btoa(this.loginForm.username).split('').reverse().join()))
                let pass = btoa(escape(btoa(this.loginForm.password).split('').reverse().join()))
                utils.setCookie('uname', name, '7d')
                utils.setCookie('ukey', pass, '7d')
              } else {
                utils.delCookie('uname')
                utils.delCookie('ukey')
              }
              this.$message({
                type: 'success',
                message: '登录成功'
              })
              this.$router.push({ path: '/account/home' })
            } else {
              this.$message({
                type: 'error',
                message: res.data.message
              })
              this.drawCode()
            }
            self.logging = false
          })
          .catch(function (error) {
            self.logging = false
            console.log(error)
          })
        } else {
          this.invalid = true
          console.log('error submit!!');
          return false;
        }
      })
    },
    drawCode () {
      this.authCode = utils.canvasCode('canvasCode')
    },
    keyDown(e) {
      e && e.keyCode === 13 && this.submitForm()
    }
  },
  mounted () {
    this.drawCode()
    if (utils.getCookie('uname') && utils.getCookie('ukey')) {
      this.loginForm.username = atob(unescape(atob(utils.getCookie('uname'))).split(',').reverse().join(''))
      this.loginForm.password = atob(unescape(atob(utils.getCookie('ukey'))).split(',').reverse().join(''))
      this.loginForm.remember = true
    }
    let codeInput = document.querySelectorAll('.el-input__inner')[2]
    codeInput.addEventListener('keydown', this.keyDown)
  }
}
</script>
<style scoped>
  .container {
    overflow: hidden
  }
	.login-form {
		width: 430px;
		margin: 8% auto 0;
		padding: 30px;
		border-radius: 5px;
		background: #fff
	}
  .canvas-code {
    float: right;
    cursor: pointer;
    border-radius: 4px;
    background-image: linear-gradient(-45deg, #111, #666, #222, #777);
  }
  .slide-fade-enter-active {
    transition: all .5s;
  }
  .slide-fade-enter, .slide-fade-leave-active {
    opacity: 0;
    transform: translateY(-30px);
  }
  .animated {
    -webkit-animation-duration: 1s;
    animation-duration: 1s;
    -webkit-animation-fill-mode: both;
    animation-fill-mode: both;
    z-index: 100
  }

  .animated.infinite {
    -webkit-animation-iteration-count: infinite;
    animation-iteration-count: infinite
  }

  .animated.hinge {
    -webkit-animation-duration: 2s;
    animation-duration: 2s
  }
  @-webkit-keyframes shake {
    0%,
    100% {
      -webkit-transform: translateX(0);
      transform: translateX(0)
    }
    10%,
    30%,
    50%,
    70%,
    90% {
      -webkit-transform: translateX(-10px);
      transform: translateX(-10px)
    }
    20%,
    40%,
    60%,
    80% {
      -webkit-transform: translateX(10px);
      transform: translateX(10px)
    }
  }

  @keyframes shake {
    0%,
    100% {
      -webkit-transform: translateX(0);
      -ms-transform: translateX(0);
      transform: translateX(0)
    }
    10%,
    30%,
    50%,
    70%,
    90% {
      -webkit-transform: translateX(-10px);
      -ms-transform: translateX(-10px);
      transform: translateX(-10px)
    }
    20%,
    40%,
    60%,
    80% {
      -webkit-transform: translateX(10px);
      -ms-transform: translateX(10px);
      transform: translateX(10px)
    }
  }

  .shake {
    -webkit-animation-name: shake;
    animation-name: shake
  }
</style>
