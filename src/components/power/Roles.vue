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
            <!-- 第一列可展开栏 -->
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
                            <el-tag>
                                {{item.authName}}
                                <i class="el-icon-caret-right"></i>
                            </el-tag>
                        </el-col>
                        <el-col :span="19">
                            <el-row v-for="(item2,index2) in item.children" :key="item2.id"
                            :class="[index2===0?'':'tagtop','vcenter']">
                                <el-col :span="7">
                                    <el-tag type="success">
                                        {{item2.authName}}
                                        <i class="el-icon-caret-right"></i>
                                    </el-tag>
                                </el-col>
                                <el-col :span="17">
                                    <el-tag type="warning"
                                    v-for="(item3) in item2.children" :key="item3.id">
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
                <template>
                    <el-button type="primary" round icon="el-icon-edit" size="mini">编辑</el-button>
                    <el-button type="danger" round icon="el-icon-delete" size="mini">删除</el-button>
                    <el-button type="warning" round icon="el-icon-setting" size="mini">分配权限</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      roleslist: []
    }
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.roleslist = res.data
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
