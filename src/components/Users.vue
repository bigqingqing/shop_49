<template>
  <div class="users">
    <!-- 面包屑导航 -->
    <el-breadcrumb class="margin" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索框 -->
    <el-input placeholder="请输入内容" v-model="query" class="input-with-select">
      <el-button slot="append" icon="el-icon-search" @click="search"></el-button>
    </el-input>
    <el-button type="success" plain @click="showadd">添加</el-button>
    <!-- 表格 -->
    <el-table border :data="tableList" style="width: 100%">
      <el-table-column prop="username" label="用户名" width="180"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="180"></el-table-column>
      <el-table-column prop="mobile" label="电话" width="180"></el-table-column>
      <el-table-column label="用户状态" width="180">
        <template v-slot:default="obj">
          <el-switch
            @change="changeState(obj.row)"
            v-model="obj.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template v-slot:default="obj">
          <el-button
            plain
            size="small"
            icon="el-icon-edit"
            type="primary"
            @click="showEditDialog(obj.row)"
          >修改</el-button>
          <el-button
            plain
            size="small"
            @click="delbtn(obj.row.id)"
            icon="el-icon-delete"
            type="danger"
          >删除</el-button>
          <el-button plain size="small" icon="el-icon-check" type="success">分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页器 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="pagesize"
      layout="  prev, pager, next,sizes, jumper,total"
      :total="this.total"
    ></el-pagination>
    <!-- 对话框 -->
    <el-dialog title="添加用户" :visible.sync="dialogVisible" width="40%" @close="closeDialog">
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input placeholder="请输入用户名" v-model="form.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input type="password" placeholder="请输入密码" v-model="form.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input placeholder="请输入邮箱" v-model="form.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input placeholder="请输入手机号" v-model="form.mobile"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="add">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改框 -->
    <el-dialog title="修改用户" :visible.sync="editVisible" width="40%">
      <el-form ref="editForm" :rules="rules" :model="editForm" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-tag type="info">{{ editForm.username }}</el-tag>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input placeholder="请输入邮箱" v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input placeholder="请输入手机" v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      dialogVisible: false,
      input: '',
      tableList: [],
      query: '',
      pagenum: 1, // 当前页
      pagesize: 4, // 一页多少一条
      total: 0,
      form: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 12,
            message: '用户名长度在 3 到 12 个字符',
            trigger: ['blur', 'change']
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 3,
            max: 12,
            message: '密码长度在 3 到 12 个字符',
            trigger: ['blur', 'change']
          }
        ],
        email: [
          {
            type: 'email',
            message: '请输入正确的邮箱地址',
            trigger: ['blur', 'change']
          }
        ],
        mobile: [
          {
            pattern: /^1[3-9]\d{9}/,
            message: '请输入正确的手机号',
            trigger: ['blur', 'change']
          }
        ]
      },
      editVisible: false,
      editForm: {
        id: '',
        username: '',
        email: '',
        mobile: ''
      }
    }
  },
  created () {
    this.getUsersList()
  },
  methods: {
    // 获取页面用户列表 ,通过ajax请求
    async getUsersList () {
      const { meta, data } = await this.$axios.get('users', {
        params: {
          query: this.query,
          pagenum: this.pagenum,
          pagesize: this.pagesize
        }
      })
      if (meta.status === 200) {
        this.tableList = data.users
        this.total = data.total
      } else {
        this.$message.error(meta.msg)
      }
    },
    // 改变状态
    async changeState (row) {
      const res = await this.$axios.put(
        `users/${row.id}/state/${row.mg_state}`
      )
      const { meta } = res
      if (meta.status === 200) {
        this.$message.success('修改成功')
      } else {
        this.$message.error(meta.msg)
      }
    },
    // 修改当前页的内容
    handleCurrentChange (val) {
      this.pagenum = val
      this.getUsersList()
    },
    // 修改每页条数
    handleSizeChange (val) {
      console.log(`每页 ${val} 条`)
      this.pagesize = val
      this.pagenum = 1
      this.getUsersList()
    },
    // 删除按钮的点事件
    async delbtn (id) {
      try {
        await this.$confirm('确认要删除嘛?', '提示', {
          type: 'warning'
        })
        const { meta } = await this.$axios.delete(`users/${id}`)
        if (meta.status === 200) {
          // 删除成功
          this.$message.success('删除成功')
          // 如果当前页只有一条, 删除了仅有的一条后, 当前页-1
          if (this.tableList.length === 1 && this.pagenum > 1) {
            this.pagenum--
          }
          // 重新渲染当前页
          this.getUsersList()
        }
      } catch (error) {
        console.log(error)
      }

      // this.$confirm('确认要删除嘛?', '提示', {
      //   type: 'warning'
      // }).then(() => {
      //   this.$axios.delete(`users/${id}`
      //   ).then(res => {
      //     console.log(res.data)
      //     const { meta } = res
      //     if (meta.status === 200) {
      //       // 删除成功
      //       this.$message.success('删除成功')
      //       // 如果当前页只有一条, 删除了仅有的一条后, 当前页-1
      //       if (this.tableList.length === 1 && this.pagenum > 1) {
      //         this.pagenum--
      //       }
      //       // 重新渲染当前页
      //       this.getUsersList()
      //     } else {
      //       // 删除失败
      //       this.$message.error(meta.msg)
      //     }
      //   })
      // }).catch(() => {
      //   this.$message({
      //     type: 'info',
      //     message: '已取消删除'
      //   })
      // })
    },
    search () {
      this.pagenum = 1
      this.getUsersList()
    },
    showadd () {
      this.dialogVisible = true
    },
    async add () {
      try {
        await this.$refs.form.validate()
        const { meta } = await this.$axios.post('users', this.form)
        console.log(meta)
        if (meta.status === 201) {
          this.$message.success(meta.msg)
          this.dialogVisible = false // 关闭对话框

          this.total++ // 手动设置总数加一
          this.pagenum = Math.ceil(this.total / this.pagesize) // 设置新加入的一条,显示的当前页
          this.getUsersList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (error) {
        console.log(error)
      }
    },
    closeDialog () {
      this.$refs.form.resetFields()
    },
    showEditDialog (row) {
      this.editVisible = true
      this.editForm = {
        username: row.username,
        email: row.email,
        mobile: row.mobile,
        id: row.id
      }
    },
    async editUser () {
      try {
        await this.$refs.editForm.validate()
        // console.log(this.editForm)
        const { meta } = await this.$axios.put(`users/${this.editForm.id}`, this.editForm)
        console.log(meta)
        if (meta.status === 200) {
          this.$message.success(meta.msg)
          // 关闭模态框
          this.editVisible = false
          // 重新渲染
          this.getUsersList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (error) {
        console.log(error)
      }
    }
  }
}
</script>
.
<style lang="scss" scoped>
.users {
  .el-breadcrumb {
    height: 40px;
    line-height: 40px;
    border-bottom: 1px solid #ccc;
  }
  .input-with-select {
    width: 400px;
    margin: 10px;
  }
}
</style>
