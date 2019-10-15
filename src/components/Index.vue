<template>
    <el-container class="index">
      <el-header> <!--头部-->
        <div class="log">
          <span class="title">小猪猪<i class="small">--多快好省</i></span>
        </div>
        <div class="content">
          小猪猪电商后台管理系统
        </div>
        <div class="logout">
          <span> 退出请看这里~</span>
          <a href="javascript:;" @click="logout">退出</a>
        </div>
      </el-header>
      <el-container>
        <el-aside width="200px"> <!-- 侧边栏-->
          <el-menu
            unique-opened
            background-color="#545c64"
            text-color="#fff"
            active-text-color="#ffd04b">
            <el-submenu index="1">
                <template slot="title">
                  <i class="el-icon-location"></i>
                  <span>用户管理</span>
                </template>
                <el-menu-item index="1-1">
                  <i class="el-icon-menu"></i>
                  <span slot="title">用户列表</span>
                </el-menu-item>
            </el-submenu>
            <el-submenu index="2">
                <template slot="title">
                  <i class="el-icon-location"></i>
                  <span>权限管理</span>
                </template>
                <el-menu-item index="2-1">
                  <i class="el-icon-menu"></i>
                  <span slot="title">角色管理</span>
                </el-menu-item>
                <el-menu-item index="2-2">
                  <i class="el-icon-menu"></i>
                  <span slot="title">权限列表</span>
                </el-menu-item>
            </el-submenu>
          </el-menu>
        </el-aside>

        <el-main>Main</el-main> <!--主页-->
      </el-container>
    </el-container>
</template>

<script>
export default {
  data () {
    return {

    }
  },
  methods: {
    logout () {
      this.$confirm('确定要退出系统嘛', '温馨提示', {
        // confirmButtonText: '确定',
        // cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$message.success('恭喜, 退出成功')
        localStorage.removeItem('token')
        this.$router.push('/login')
      }).catch(() => {
        this.$message('取消退出')
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.index{
  height: 100%;
  .el-header{
    display: flex;
    background-color: #d8d8d8;
    .log{
      width: 180px;
      padding: 0;
      height: 60px;
      .title{
      font-size: 24px;
      font-weight: 700;
      color: #fff;
      line-height: 60px;
      }
      .small{
      font-size: 12px;
      color: #545c64;
    }
    }
    .content{
      flex: 1;
      text-align: center;
      font-size: 30px;
      line-height: 60px;
      color: #fff;
    }
    .logout{
      line-height: 60px;
      color: #999;
      a{
        color: orange;
      }

    }
  }
  .el-aside{
    background-color: #545c64;
    .el-menu{
      border:none;
    }
  }
  .el-main{
    background-color: #fff;
  }
}
</style>
