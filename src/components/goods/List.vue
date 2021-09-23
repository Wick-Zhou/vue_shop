<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 搜索框和添加按钮 -->
      <el-row :gutter="30">
        <el-col :span="7">
          <el-input placeholder="搜索商品" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="addGood">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 用户数据列表区 -->
      <el-table  :data="goodslist" style="width: 100%" stripe>
        <el-table-column type="index" label="#" width="80px"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称" width="250px"></el-table-column>
        <el-table-column prop="goods_price" label="商品价格(元)" width="180px" align="center"></el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="180px" align="center"></el-table-column>
        <el-table-column label="创建时间" width="180px">
          <template slot-scope="scope">{{scope.row.add_time|dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作" align="center">
          <template slot-scope="scope">
            <el-button type="primary" round icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)"></el-button>
            <el-button type="danger" round icon="el-icon-delete" size="mini" @click="removeGood(scope.row.goods_id)"></el-button>
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
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      goodslist: []
    }
  },
  methods: {
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.goodslist = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange (val) {
      this.queryInfo.pagesize = val
      this.getGoodsList()
    },
    handleCurrentChange (val) {
      this.queryInfo.pagenum = val
      this.getGoodsList()
    },
    async removeGood (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => {
        return err
      })
      if (confirmResult === 'cancel') return
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.getGoodsList()
    },
    addGood () {
      this.$router.push('addgood')
    },
    showEditDialog (row) {

    }
  },
  created () {
    this.getGoodsList()
  }
}
</script>

<style lang="less" scoped>
  .el-card{
    margin-top: 20px;
  }
</style>
