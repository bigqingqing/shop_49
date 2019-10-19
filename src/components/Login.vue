<template>
  <div class="login">
    <img src="../assets/avatar.jpg" alt />
    <el-form
      ref="form"
      :model="form"
      label-width="80px"
      :label-position="labelPosition"
      :rules="rules"
      status-icon
    >
      <el-form-item label="用户名" prop="username">
        <el-input v-model="form.username" @keyup.enter.native="login" placeholder="请输入用户名"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input v-model="form.password" @keyup.enter.native="login" type="password" placeholder="请输入密码"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button class="btn-r" @click="login" type="primary">登录</el-button>
        <el-button @click="reset">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>

export default {
  data () {
    return {
      form: {
        username: '',
        password: ''
      },
      labelPosition: 'left',
      rules: {
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: ['blur', 'change']
          },
          {
            min: 3,
            max: 12,
            message: '长度在 3 到 12 位的用户名',
            trigger: ['blur', 'change']
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: ['blur', 'change']
          },
          {
            min: 3,
            max: 12,
            message: '长度在 3 到 12 位的密码',
            trigger: ['blur', 'change']
          }
        ]
      }
    }
  },
  methods: {
    reset () {
      console.log(1)
      this.$refs.form.resetFields()
    },
    async login () {
      try {
        await this.$refs.form.validate()
        const { meta, data } = await this.$axios({
          method: 'post',
          url: 'login',
          data: this.form
        })
        localStorage.setItem('token', data.token) // 将data里的token拿出来存在localStorage
        if (meta.status === 200) {
          this.$message({
            message: meta.msg,
            type: 'success'
          })
          this.$router.push('/Index')
        }
      } catch (error) {
        console.log(error)
      }
    //   this.$refs.form.validate((isValid, obj) => {
    //     // isValid 是否通过校验 obj 未通过校验的信息
    //     if (!isValid) return
    //     // 通过校验了, 该发ajax了
    //     this.$axios({
    //       method: 'post',
    //       url: 'login',
    //       data: this.form
    //     }).then(res => {
    //       // console.log(res)
    //       // eslint-disable-next-line no-unused-vars
    //       const { meta, data } = res // 解构出data
    //       localStorage.setItem('token', data.token) // 将data里的token拿出来存在localStorage
    //       if (meta.status === 200) {
    //         this.$message({
    //           message: meta.msg,
    //           type: 'success'
    //         })
    //         this.$router.push('/Index')
    //       } else {
    //         this.$message({
    //           message: meta.msg,
    //           type: 'error'
    //         })
    //       }
    //     })
    //   })
    }
  }
}
</script>
<style lang="scss" scoped>
.login {
  height: 100%;
  width: 100%;
  background-color: #2d434c;
  overflow: hidden;
  position: relative;
  img {
    border-radius: 50%;
    border: 5px solid #fff;
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    top: 70px;
  }
  .el-form {
    width: 400px;
    background-color: #fff;
    margin: 200px auto;
    border-radius: 20px;
    padding: 75px 30px 25px;
    .btn-r {
      margin-right: 120px;
    }
  }
}
</style>
