<template>
    <div class="login_container">
      <div class="login_box">
        <!-- //头像区域 -->
        <div class="avatar_box">
            <img src="../assets/logo.png" alt="">
        </div>
        <!-- //登陆表区域 -->
        <el-form ref="loginFormReset" :model="login_form" :rules="login_rules" label-width="0px" class="login_form">
            <!-- 用户名 -->
            <el-form-item prop="username">
            <el-input v-model="login_form.username" prefix-icon="el-icon-user-solid"></el-input>
            </el-form-item>
            <!-- 密码 -->
            <el-form-item prop="password">
            <el-input v-model="login_form.password" prefix-icon="el-icon-lock" type="password"></el-input>
            </el-form-item>
            <!-- 按钮区域 -->
            <el-form-item class="btns">
              <el-button type="primary" @click="login">登录</el-button>
              <el-button type="info" @click="resetForm">重置</el-button>
            </el-form-item>
        </el-form>
      </div>
    </div>
</template>

<script>

export default {
  data() {
    return {
      login_form: {
        username: 'admin',
        password: '123456'
      },
      login_rules: {
        username: [
          { required: true, message: '请输入您的名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入您的密码', trigger: 'blur' },
          { min: 6, max: 12, message: '长度在 6 到 12 个数字', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetForm() {
      this.$refs.loginFormReset.resetFields()
    },
    login() {
      this.$refs.loginFormReset.validate(async valid => {
        /* valid验证为false，退出 */
        if (!valid) return
        /* 验证成功，向服务端发起（axios)请求 */
        const { data: res } = await this.$http.post('login', this.login_form)
        if (res.meta.status !== 200) return this.$message.error('登录失败！')
        this.$message.success('登陆成功！')
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
    // login(){
    //     this.$refs.loginFormReset.validate(valid=>{
    //         if(!valid) return this.$message.error("请您正确填写账号或密码！")
    //         var a = parseInt(this.login_form.password)
    //         if( a !== 88888888) return this.$message.error("登陆失败！您的密码不正确！")
    //         this.$message.success("登录成功！")
    //         window.sessionStorage.setItem('token', "zheshiyanzhengmashuju")
    //         this.$router.push('/home')
    //     })

    // }
  }
}

</script>
<style lang="less" scoped>
.login_container{
    background-color: #2b4b6b;
    height: 100%;
}
.login_box{
    width: 450px;
    height: 300px;
    background-color: #fff;
    border-radius: 3px;
    position: absolute;
    left:50%;
    top:50%;
    transform: translate(-50%,-50%);

    .avatar_box{
        height: 130px;
        width: 130px;
        border: solid 1px #eee;
        border-radius: 50%;
        padding: 10px;
        box-shadow: 0 0 10 #ddd;
        position: absolute;
        left: 50%;
        transform: translate(-50%,-50%);
        background-color: #fff;
        img{
            height: 100%;
            width: 100%;
            border-radius: 50%;
            background-color: #eee;

        }
    }
}

.login_form{
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 20px;
    box-sizing: border-box;

}
.btns{
    display: flex;
    justify-content: flex-end;
    transform: translateX(-35%);
}

</style>
