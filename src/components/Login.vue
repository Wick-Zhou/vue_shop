<template>
  <div class="login-container">
      <div class="login-box">
          <!-- 头像盒子 -->
          <div class="avatar-box">
              <img src="../assets/logo.png" alt="">
          </div>
          <!-- 登录表单区域 -->
          <el-form label-width="80px" class="login-form">
            <el-form-item label="账号">
                <el-input prefix-icon="el-icon-user"></el-input>
            </el-form-item>
            <el-form-item label="密码">
                <el-input prefix-icon="el-icon-lock"></el-input>
            </el-form-item>
            <!-- 登录 重置按钮 -->
            <el-row class="btns">
                    <el-button type="primary">登录</el-button>
                    <el-button type="danger">重置</el-button>
            </el-row>
          </el-form>

      </div>
  </div>
</template>

<script>
export default {

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
