<template>
  <div class="login-container">
      <div class="login-box">
          <!-- 头像盒子 -->
          <div class="avatar-box">
              <img src="../assets/logo.png" alt="">
          </div>
          <!-- 登录表单区域 -->
          <el-form :model="loginForm" ref="loginFormRef" label-width="80px" class="login-form" :rules="loginFormRules">
            <el-form-item label="账号" prop="username">
                <el-input v-model="loginForm.username" prefix-icon="el-icon-user"></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="password">
                <el-input v-model="loginForm.password" prefix-icon="el-icon-lock" show-password></el-input>
            </el-form-item>
            <!-- 登录 重置按钮 -->
            <el-row class="btns">
                <el-button type="primary" @click="login">登录</el-button>
                <el-button type="danger" @click="resetLoginForm">重置</el-button>
            </el-row>
          </el-form>

      </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        username: [
          { required: true, message: '请输入登录名称', trigger: 'blur' },
          { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]
      }

    }
  },
  methods: {
    resetLoginForm () {
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      this.$refs.loginFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success(res.meta.msg)
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
  }

}
</script>

<style lang="less" scoped>
.login-container{
    background-color: #2b4b6b;
    height: 100%;
}
.login-box{
    width: 450px;
    height: 300px;
    background-color: #fff;
    border-radius: 10px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);

    .avatar-box{
        height: 100px;
        width: 100px;
        img{
            height: 100%;
            width: 100%;
            border:1px,solid, #eee;
            border-radius: 50px;
        }
        border: 1px,solid,#eee;
        border-radius: 50px;
        position: absolute;
        left: 50%;
        transform: translate(-50%,-50%);
        background-color: #2b4b6b;

    }

        .logininfo-box{
            height: 100px;
            margin-left: 10px;
            margin-top: 200px;
        }

        .login-form{
            position: absolute;
            bottom: 0px;
            width: 100%;
            padding: 0 15px 0 0;
            box-sizing: border-box;
            transform: translate(-20px,0);
        }

        .btns{
            display: flex;
            justify-content: flex-end;
            margin-bottom: 10px;
        }
}
</style>
