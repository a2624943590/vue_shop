<template>
  <div class="login_container">
    <div class="login_box">
      <div class="aviator_box">
        <img src="../assets/logo.png" alt="">
      </div>
    <!--      这是登录表单区-->
      <el-form ref="loginFromRef" :rules="loginFromRules" :model="loginForm" label-width="0px" class="login_item">
     <!--        用户名-->
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
     <!--    密码-->
        <el-form-item prop="password">
          <el-input type="password" v-model="loginForm.password" prefix-icon="iconfont icon-3702mima"></el-input>
        </el-form-item>
      <!--        -->
        <el-form-item class="batons">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetForm">重置</el-button>
        </el-form-item>
      </el-form>
     </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data () {
    return {
      // 表单数据绑定
      loginForm: {
        username: '',
        password: ''
      },
      loginFromRules: {
        username: [
          { required: true, message: '请输入登录用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 6 到 10 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetForm () {
      // console.log(this)
      this.$refs.loginFromRef.resetFields()
    },
    login () {
      this.$refs.loginFromRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        console.log(res)
        if (res.meta.status !== 200) return this.$message.error('登陆失败')
        this.$message.success('登陆成功')

        // 登录成功的token，保存在客户端sessionStorage中，只是在网站打开旗舰生效，不适用localStorage
        window.sessionStorage.setItem('token', res.data.token)
        // 通过编程式导航跳转到后台的主页
        await this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less"  scoped>
 .login_container{
    background-color: #2c3e50;
    height: 100%;
 }
 .login_box{
    width: 450px;
    height: 300px;
    background-color: #fff;
    border-radius: 3px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
 }
 .aviator_box{
   height: 130px;
   width: 130px;
   border:1px solid #eee;
   border-radius: 50%;
   padding: 10px;
   box-shadow: 0 0 10px #ddd;
   position: absolute;
   left: 50%;
   transform: translate(-50%,-50%);
   background-color: #fff;
   img{
     width: 100%;
     height: 100%;
     border-radius: 50%;
     background-color: #eeeeee;
   }
 }
 .batons{
    display: flex;
    justify-content: flex-end;
 }
 .login_item{
   position: absolute;
   bottom: 0;
   width: 100%;
   padding: 0 20px;
   box-sizing: border-box;
 }
</style>
