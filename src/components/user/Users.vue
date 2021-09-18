<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>用户管理</el-breadcrumb-item>
    <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区 -->
    <el-card class="box-card">
      <!-- 搜索框和添加按钮 -->
      <el-row :gutter="30">
        <el-col :span="7">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="dialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户数据列表区 -->
      <el-table  :data="userlist" style="width: 100%">
        <el-table-column type="index" label="#" width="80px"></el-table-column>
        <el-table-column prop="id" label="ID" width="100px"></el-table-column>
        <el-table-column prop="username" label="姓名" width="150px"></el-table-column>
        <el-table-column prop="role_name" label="角色" width="150px"></el-table-column>
        <el-table-column prop="mobile" label="电话" width="180px"></el-table-column>
        <el-table-column prop="email" label="邮箱" width="180px"></el-table-column>
        <el-table-column label="状态" width="120px">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              active-color="#13ce66"
              inactive-color="#909399"
              @change="userStateChange(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="primary" round icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)"></el-button>
            <el-button type="danger" round icon="el-icon-delete" size="mini" @click="removeUser(scope.row.id)"></el-button>
            <el-tooltip class="item" effect="dark" content="编辑权限" placement="top" :enterable="false">
              <el-button type="warning" round icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1,2,3,4,5,6,7,8,9,10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加用户的对话框 -->
    <el-dialog
    title="添加用户"
    :visible.sync="dialogVisible"
    width="50%"
    @close="resetAddUserForm">
      <el-form :model="addUserForm" ref="addUserFormRef" label-width="80px" :rules="addUserFormRules">
        <el-form-item label="账号" prop="username">
            <el-input v-model="addUserForm.username" prefix-icon="el-icon-user"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
            <el-input v-model="addUserForm.password" prefix-icon="el-icon-lock" show-password></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
            <el-input v-model="addUserForm.email" prefix-icon="el-icon-s-promotion"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
            <el-input v-model="addUserForm.mobile" prefix-icon="el-icon-phone"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改 编辑用户信息的对话框 -->
    <el-dialog
      title="修改信息"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="resetEditForm">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef">
        <el-input
          :placeholder="editForm.username"
          :disabled="true">
        </el-input>
        <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email" prefix-icon="el-icon-s-promotion"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
            <el-input v-model="editForm.mobile" prefix-icon="el-icon-phone"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    // 验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) {
        return cb()
      }
      // 返回一个错误提示
      cb(new Error('请输入合法的邮箱'))
    }
    // 验证手机号码的规则
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) {
        return cb()
      }
      // 返回一个错误提示
      cb(new Error('请输入合法的手机号码'))
    }
    return {
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 每页多少数据
        pagesize: 4
      },
      userlist: [],
      total: 0,
      dialogVisible: false,
      editDialogVisible: false,
      addUserForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editForm: {
        username: '',
        email: '',
        mobile: ''
      },
      // 添加用户时的表单验证规则
      addUserFormRules: {
        username: [
          { required: true, message: '请输入登录名称', trigger: 'blur' },
          { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, message: '邮箱格式不正确，请重新输入', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入电话', trigger: 'blur' },
          { validator: checkMobile, message: '邮箱格式不正确，请重新输入', trigger: 'blur' }
        ]
      },
      // 编辑用户时的表单验证规则
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, message: '邮箱格式不正确，请重新输入', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入电话', trigger: 'blur' },
          { validator: checkMobile, message: '邮箱格式不正确，请重新输入', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      // console.log(res)
      this.userlist = res.data.users
      this.total = res.data.total
    },
    // 改变每页多少条数据
    handleSizeChange (val) {
      this.queryInfo.pagesize = val
      this.getUserList()
    },
    // 跳转到多少页
    handleCurrentChange (val) {
      this.queryInfo.pagenum = val
      this.getUserList()
    },
    // 修改用户列表中的状态并发送到数据库修改
    async userStateChange (userInfo) {
      console.log(userInfo)
      const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      if (res.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
    },
    // 关闭添加用户框时清空表单中的数据
    resetAddUserForm () {
      this.$refs.addUserFormRef.resetFields()
    },
    // 清空编辑表单的验证提示
    resetEditForm () {
      this.$refs.editFormRef.resetFields()
    },
    // 添加用户
    addUser () {
      this.$refs.addUserFormRef.validate(async valid => {
        if (valid) {
          const { data: res } = await this.$http.post('users', this.addUserForm)
          if (res.meta.status === 201) {
            this.$message.success(res.meta.msg)
            this.dialogVisible = false
            this.getUserList()
          }
        }
      })
    },
    // 点击编辑按钮显示编辑框
    showEditDialog (userInfo) {
      this.editDialogVisible = true
      this.editForm = userInfo
    },
    // 修改用户信息并提交到数据库更改
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        if (valid) {
          const { data: res } = await this.$http.put(`users/${this.editForm.id}`, { email: this.editForm.email, mobile: this.editForm.mobile })
          if (res.meta.status === 200) {
            this.$message.success(res.meta.msg)
            this.editDialogVisible = false
            this.getUserList()
          }
        }
      })
    },
    async removeUser (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => {
        return err
      })
      // console.log(confirmResult)
      // 点击确定返回字符串 confirm
      // 点击取消返回字符串 cancel
      if (confirmResult === 'cancel') {
        return this.$message.info('已取消删除')
      }

      const { data: res } = await this.$http.delete(`users/${id}`)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.getUserList()
    }

  },
  created () {
    this.getUserList()
  }

}
</script>

<style lang="less" scoped>
  .el-card{
    margin-top: 20px;
    .el-table{
      margin-top: 20px;
    }
    .el-pagination{
      margin-top: 20px;
    }
  }
</style>
