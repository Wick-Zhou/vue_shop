<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
        <el-button type="primary">添加角色</el-button>
        <el-table :data="roleslist" style="width: 100%" border stripe>
            <!-- 第一列可展开栏、、、 -->
            <!-- 展开栏的整体布局
                    三层权限一共分为2列(el一共分为24份)：
                        第一列为第一层权限(占5份)
                        第二列为2、3层权限(占剩余的19份)
                            把这部分又分为24份
                            第二列当中的第二层权限分到第一列(占其中的7份)
                            第二列当中的第三层权限分到第二列(占其中17份)
            -->
            <el-table-column type="expand">
                <template slot-scope="scope">
                    <el-row v-for="(item,index) in scope.row.children" :key="item.id"
                    :class="['tagbottom',index===0?'tagtop':'','vcenter']">
                        <el-col :span="5">
                            <el-tag closable @close="removeRight(scope.row,item.id)">
                                {{item.authName}}
                                <i class="el-icon-caret-right"></i>
                            </el-tag>
                        </el-col>
                        <el-col :span="19">
                            <el-row v-for="(item2,index2) in item.children" :key="item2.id"
                            :class="[index2===0?'':'tagtop','vcenter']">
                                <el-col :span="7">
                                    <el-tag type="success" closable @close="removeRight(scope.row,item2.id)">
                                        {{item2.authName}}
                                        <i class="el-icon-caret-right"></i>
                                    </el-tag>
                                </el-col>
                                <el-col :span="17">
                                    <el-tag type="warning"
                                    v-for="(item3) in item2.children" :key="item3.id"
                                    closable @close="removeRight(scope.row,item3.id)">
                                        {{item3.authName}}
                                        <i class="el-icon-caret-right"></i>
                                    </el-tag>
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>
                </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column prop="roleName" label="角色名称"></el-table-column>
            <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
            <el-table-column prop="level" label="操作">
                <template slot-scope="scope">
                    <el-button type="primary" round icon="el-icon-edit" size="mini">编辑</el-button>
                    <el-button type="danger" round icon="el-icon-delete" size="mini">删除</el-button>
                    <el-button type="warning" round icon="el-icon-setting" size="mini" @click="showSetRigthsDialog(scope.row)">分配权限</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>
    <!-- 选择 编辑权限对话框 -->
    <el-dialog
    title="权限管理"
    :visible.sync="dialogVisible"
    width="50%"
    @close="setRightDialogClosed">
        <!-- 树形结构 -->
        <el-tree
        :data="roleslist"
        :props="treeProps"
        default-expand-all
        node-key="id"
        :default-checked-keys="defKeys"
        show-checkbox
        ref="treeRef">
        </el-tree>
        <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="confirmSelectRights">确 定</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      roleslist: [],
      dialogVisible: false,
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      defKeys: [],
      // 分配角色id供之后确认权限时使用
      roleId: ''
    }
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.roleslist = res.data
    },
    async removeRight (role, id) {
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => {
        return err
      })
      if (confirmResult === 'cancel') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${id}`)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      //   this.getRolesList() 会导致折叠页面收回 效果不好
      role.children = res.data
    },
    async showSetRigthsDialog (role) {
      this.roleId = role.id
      this.dialogVisible = true
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.roleslist = res.data
      this.getLeafKeys(role, this.defKeys)
    },
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    setRightDialogClosed () {
      this.defKeys = []
      this.getRolesList()
    },
    async confirmSelectRights () {
      const rightArr = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const rightStr = rightArr.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: rightStr })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.dialogVisible = false
      this.getRolesList()
    }
  },
  created () {
    this.getRolesList()
  }

}
</script>

<style lang="less" scoped>
    .el-card{
        margin-top: 20px;
        .el-table{
            margin-top: 20px;
        }
    }
    .el-tag{
        margin: 10px;
    }
    .tagtop{
        border-top: 1px solid #eee;
    }
    .tagbottom{
        border-bottom: 1px solid #eee;
    }
    .vcenter{
        display: flex;
        align-items: center;
    }
</style>
