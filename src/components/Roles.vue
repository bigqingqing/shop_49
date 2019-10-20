<template>
  <div class="roles">
    <!-- 面包屑导航 -->
    <el-breadcrumb class="margin" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 添加按钮 -->
    <el-button size="small" class="mar-btn" type="success" @click="showAddroles" plain>添加角色</el-button>
    <!-- 表格 -->
    <el-table :data="rolesList">
      <el-table-column type="expand">
        <template v-slot:default="{row}">
          <p v-if="row.children.length===0">暂无权限</p>
          <el-row class="l1-style"  v-for="l1 in row.children" :key="l1.id">
            <el-col :span="4">
              <!-- 一级权限 l1 -->
              <el-tag @close="delright(row,l1.id)"  closable> {{l1.authName}}</el-tag>
              <i class="el-icon-arrow-right "></i>
            </el-col>
            <el-col :span="20">
              <!-- 二级权限 -l2 -->
              <el-row class="l2style" v-for="l2 in l1.children" :key="l2.id">
                <el-col :span="4">
                  <el-tag @close="delright(row,l2.id)"  closable type="success">
                  {{l2.authName}}
                  </el-tag>
                   <i class="el-icon-arrow-right "></i>
                  </el-col>
                <el-col :span="20">
                  <el-tag @close="delright(row,l3.id)" class="l3" closable type="warning" v-for="l3 in l2.children" :key="l3.id">{{l3.authName}}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>

        </template>
      </el-table-column>
      <el-table-column type="index" width="50"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作">
        <template v-slot:default="{row}">
          <el-button plain size="small" type="primary" icon="el-icon-edit" @click="showEidtroles(row)"></el-button>
          <el-button plain size="small" type="danger" icon="el-icon-delete" @click="delbtn(row.id)"></el-button>
          <el-button plain size="small" type="success" icon="el-icon-check" @click="showAssignDialog(row)">分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 修改框 -->
    <el-dialog title="修改用户" :visible.sync="editVisible" width="40%">
      <el-form ref="editForm" :rules="rules" :model="editForm" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input placeholder="请输入角色名称" v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="描述" prop="roleDesc">
          <el-input placeholder="请输入描述" v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="eidtroles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加用户对话框 -->
    <el-dialog title="添加用户" :visible.sync="addVisible" width="40%" @close="closeDialog">
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input placeholder="请输入角色名称" v-model="form.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input placeholder="请输入角色描述" v-model="form.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addroles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="assignVisible"
      width="40%">
      <!-- 树形结构 -->
      <el-tree
        default-expand-all
        show-checkbox
        node-key="id"
        ref="tree"
        :data="assigndata"
        :props="defaultProps"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="assignRights">确 定</el-button>
      </span>
    </el-dialog>

  </div>

</template>

<script>
export default {
  data () {
    return {
      roleId: '',
      rolesList: [],
      editVisible: false,
      editForm: {
        id: '',
        roleName: '',
        roleDesc: ''
      },
      addVisible: false,
      form: {
        roleName: '',
        roleDesc: ''
      },
      assignVisible: false,
      assigndata: [],
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      rules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 3, max: 12, message: '长度在 3 到 12个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 3, max: 50, message: '长度在 3 到 50个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getrolesList()
  },
  methods: {
    async getrolesList () {
      const { data, meta } = await this.$axios.get('roles')
      // console.log(data, meta)
      if (meta.status === 200) {
        this.rolesList = data
        // console.log(this.rolesList)
      } else {
        this.$message.error(meta.msg)
      }
    },
    async delright (row, rightId) {
      const { data, meta } = await this.$axios.delete(`roles/${row.id}/rights/${rightId}`)
      // console.log(data, meta)
      if (meta.status === 200) {
        this.$message.success(meta.msg)
        row.children = data
      } else {
        this.$message.error(meta.msg)
      }
    },
    async delbtn (id) {
      try {
        await this.$confirm('确认要删除嘛?', '提示', {
          type: 'warning'
        })
        await this.$axios.delete(`roles/${id}`)
        this.getrolesList()
      } catch (error) {
        console.log(error)
      }
    },
    showEidtroles (row) {
      this.editVisible = true
      this.editForm = {
        roleName: row.roleName,
        roleDesc: row.roleDesc,
        id: row.id
      }
      console.log(this.editForm.id)
    },
    async eidtroles () {
      try {
        await this.$refs.editForm.validate()
        const { roleName, roleDesc, id } = this.editForm
        const { meta } = await this.$axios.put(`roles/${id}`, {
          roleName,
          roleDesc
        })
        if (meta.status === 200) {
          this.$message.success(meta.msg)
          // 关闭模态框
          this.editVisible = false
          // 重新渲染
          this.getrolesList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (error) {
        console.log(error)
      }
    },
    showAddroles () {
      this.addVisible = true
    },
    async addroles () {
      try {
        await this.$refs.form.validate()
        const { meta } = await this.$axios.post('roles', this.form)
        // console.log(meta)
        if (meta.status === 201) {
          this.$message.success(meta.msg)
          this.addVisible = false // 关闭对话框
          this.getrolesList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (error) {
        this.$message.error(error)
      }
    },
    closeDialog () {
      this.$refs.form.resetFields()
    },
    async showAssignDialog (row) {
      this.roleId = row.id
      this.assignVisible = true
      const { data, meta } = await this.$axios.get('rights/tree')
      if (meta.status === 200) {
        // 成功了, 将数据存起来
        this.assigndata = data
        console.log(data)
        const ids = []
        row.children.forEach(l1 => {
          l1.children.forEach(l2 => {
            l2.children.forEach(l3 => {
              ids.push(l3.id)
            })
          })
        })
        this.$refs.tree.setCheckedKeys(ids)
      } else {
        this.$message.error(meta.msg)
      }
    },
    async assignRights () {
      console.log(1)
      const ids = this.$refs.tree.getCheckedKeys() // 全选的
      const halfs = this.$refs.tree.getHalfCheckedKeys() // 半选的
      const rids = [...ids, ...halfs].join(',')
      const { meta } = await this.$axios.post(`roles/${this.roleId}/rights`, { rids })
      if (meta.status === 200) {
        // 提示分配成功
        this.$message.success(meta.msg)
        // 关闭对话框
        this.assignVisible = false
        // 重新渲染
        this.getrolesList()
      } else {
        this.$message.error(meta.msg)
      }
    }
  }

}

</script>

<style lang="scss" scoped>
.roles {
  .mar-btn {
    margin: 10px;
  }
  .l1-style{
    margin-bottom: 10px;
    border-bottom: 1px dotted #ccc;
  }
  .l2style{
    margin-bottom: 10px
  }
  .l3{
    margin-right: 5px;
    margin-top: 5px;
  }
}
</style>
