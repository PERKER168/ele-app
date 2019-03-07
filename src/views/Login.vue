<template>
  <div class="login">
    <div class="logo">
      <img src="../assets/logo.jpg" alt="my login image">
    </div>
    
    <!-- 手机号码 -->
    <InputGrounp 
     type="number" 
     v-model="phone" 
     placeholder="手机号码" 
     :btnTitle="btnTitle" 
     :disabled="disabled" 
     :error="errors.phone"
     @btnClick="getVerifyCode"/>
    <!-- 验证码 -->
    <InputGrounp type="number" v-model="verifyCode" placeholder="验证码" :error="errors.code"/>
    
    <!-- 用户协议 -->
    <div class="login-des">
      <p>
        新用户登录即自动注册，表示已同意<span>用户协议</span>
      </p>
    </div>

    <!-- 登陆按钮 -->
    <div class="login-btn">
      <button :disabled="isClick" @click="handleLogin">登录</button>
    </div>
  </div>
</template>

<script>
import InputGrounp from '../components/InputGroup'
import { setInterval, clearInterval } from 'timers';
export default {
  name:'login',
  data() {
    return {
      phone: '',
      verifyCode: '',
      errors: {},
      btnTitle: '获取验证码',
      disabled: false
    }
  },
  methods: {
    handleLogin(){
      // 取消错误提醒
      this.errors = {};
      // 发送请求
      this.$axios.post('/api/posts/sms_back',{
        phone: this.phone,
        code: this.verifyCode
      }).then(res => {
        console.log(res);
        // 检验成功，设置登陆状态并且跳转
        localStorage.setItem('ele_login',true);
        this.$router.push('/');
      }).catch(err => {
        // 返回错误信息
        this.errors = {
          code: err.response.data.msg
        }
      });
    },
    // 获取验证码
    getVerifyCode(){
      if (this.validatePhone()) {
        this.validateBtn();
        // 发送网络请求
        this.$axios.post('/api/posts/sms_send',{
          tpl_id:'138893',    // 模板id
          key:'5c20adf8d6155f502e4f38f77b4d6afe', // AppKey
          phone:this.phone   // 接收验证码的手机号码
        }).then(res => {
          console.log(res);
        });
      }
    },
    validateBtn(){
      let time = 60;
      let timer = setInterval(() => {
        if (time == 0) {
          clearInterval(timer);
          this.btnTitle = '获取验证码';
          this.disabled = false;
        }else{
          // 倒计时
          this.btnTitle = time + '秒后重试';
          this.disabled = true;
          time--;
        }
      },1000);
    },
    validatePhone(){
      // 验证手机号码
      if (!this.phone) {
        this.errors = {
          phone:'手机号码不能为空'
        }
        return false;
      }else if(!/^1[345678]\d{9}$/.test(this.phone)) {
        this.errors = {
          phone: '请填写正确的手机号码'
        }
        return false;
      }else{
        this.errors = {};
        return true;
      }
    }
  },
  computed: {
    isClick(){
      if(!this.phone || !this.verifyCode) return true;
      else return false;
    }
  },
  components: {
    InputGrounp
  }
}
</script>

<style>
.login {
  width: 100%;
  height: 100%;
  padding: 30px;
  box-sizing: border-box;
  background: #fff;
}

.logo {
  text-align: center;
}
.logo img {
  width: 150px;
}
.text-group,
.login-des,
.login-btn {
  margin-top: 20px;
}
.login-des {
  color: #aaa;
  line-height: 22px;
}
.login-des span {
  color: #4d90fe;
}
.login-btn button {
  width: 100%;
  height: 40px;
  background-color: #48ca38;
  border-radius: 4px;
  color: white;
  font-size: 14px;
  border: none;
  outline: none;
}
.login-btn button[disabled] {
  background-color: #8bda81;
}
</style>
