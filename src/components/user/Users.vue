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
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary">添加用户</el-button>
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
        <el-table-column prop="" label="操作">
          <template>
            <el-button type="primary" round icon="el-icon-delete" size="mini"></el-button>
            <el-button type="danger" round icon="el-icon-delete" size="mini"></el-button>
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
  </div>
</template>

<script>
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 每页多少数据
        pagesize: 5
      },
      userlist: [],
      total: 0
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
    async userStateChange (userInfo) {
      console.log(userInfo)
      const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      if (res.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
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
