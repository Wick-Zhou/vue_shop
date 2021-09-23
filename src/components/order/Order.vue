<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="30">
        <el-col :span="7">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table  :data="orderlist" style="width: 100%">
        <el-table-column type="index" label="#" ></el-table-column>
        <el-table-column prop="order_number" label="订单编号" width="220px"></el-table-column>
        <el-table-column prop="order_price" label="订单价格"></el-table-column>
        <el-table-column label="是否付款" >
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.order_pay===1">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货"  align="center"></el-table-column>
        <el-table-column label="下单时间" >
          <template slot-scope="scope">{{scope.row.create_time|dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="primary" round icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)"></el-button>
            <el-button type="success" round icon="el-icon-location" size="mini" @click="showProgressBox(scope.row.id)"></el-button>
          </template>
        </el-table-column>
      </el-table>
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
    <el-dialog
    title="修改地址"
    :visible.sync="editAddressDialogVisible"
    width="50%"
    @close="resetEditAddressForm">
      <el-form :model="editAddressForm" ref="editAddressFormRef" label-width="80px" :rules="editAddressFormRules">
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader :options="cityData" v-model="editAddressForm.address1"></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="editAddressForm.address2" prefix-icon="el-icon-lock"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editAddressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editAddress">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog
    title="物流查询"
    :visible.sync="progressDialogVisible"
    width="50%">
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata'
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 4
      },
      total: 0,
      orderlist: [],
      editAddressDialogVisible: false,
      editAddressForm: {
        address1: [],
        address2: ''
      },
      editAddressFormRules: {
        address1: [
          { required: true, message: '请选择地址', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      progressDialogVisible: false,
      progressInfo: []
    }
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.orderlist = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange (val) {
      this.queryInfo.pagesize = val
      this.getOrderList()
    },
    handleCurrentChange (val) {
      this.queryInfo.pagenum = val
      this.getOrderList()
    },
    showEditDialog () {
      this.editAddressDialogVisible = true
    },
    resetEditAddressForm () {
      this.$refs.editAddressFormRef.resetFields()
    },
    async showProgressBox () {
      this.progressDialogVisible = true
      // const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      // if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      // this.progressInfo = res.data
      // console.log(this.progressInfo)
    },
    editAddress () {

    }
  },
  created () {
    this.getOrderList()
  }
}
</script>

<style lang="less" scoped>
  .el-card{
    margin-top: 20px;
  }
</style>
