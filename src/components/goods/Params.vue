<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert title="只允许选择三级分类设置相关参数" type="warning" show-icon :closable="false">
      </el-alert>
      <el-row style="margin:20px 0">
        <span>选择商品分类：</span>
        <el-cascader
          v-model="selectKeys"
          :options="catelist"
          :props="cascaderProps"
          @change="handleChange"
          clearable>
        </el-cascader>
      </el-row>
      <template>
        <el-tabs v-model="activeName" type="card" @tab-click="handleClick">
          <el-tab-pane label="动态参数" name="many">
            <el-button type="primary" :disabled="isBtnDisabled" @click="showAddDialog">添加参数</el-button>
            <el-table  :data="manyData" style="width: 100%">
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <el-tag v-for="(item,index) in scope.row.attr_vals" :key="index" closable @close="handleClose(index,scope)">{{item}}</el-tag>
                  <el-input
                    class="input-new-tag"
                    v-if="scope.row.inputVisible"
                    v-model="scope.row.inputValue"
                    ref="saveTagInputRef"
                    size="small"
                    @keyup.enter.native="handleInputConfirm(scope)"
                    @blur="handleInputConfirm(scope)"
                  >
                  </el-input>
                  <el-button v-else class="button-new-tag" size="small" @click="showInput(scope)">+ New Tag</el-button>
                </template>
              </el-table-column>
              <el-table-column type="index" label="#"></el-table-column>
              <el-table-column label="姓名" prop="attr_name"></el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button type="primary" round icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)">编辑</el-button>
                  <el-button type="danger" round icon="el-icon-delete" size="mini" @click="removeParams(scope.row)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
          <el-tab-pane label="静态属性" name="only">
            <el-button type="primary" :disabled="isBtnDisabled" @click="showAddDialog">添加属性</el-button>
            <el-table  :data="onlyData" style="width: 100%">
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <el-tag v-for="(item,index) in scope.row.attr_vals" :key="index" closable @close="handleClose(index,scope)">{{item}}</el-tag>
                  <el-input
                    class="input-new-tag"
                    v-if="scope.row.inputVisible"
                    v-model="scope.row.inputValue"
                    ref="saveTagInputRef"
                    size="small"
                    @keyup.enter.native="handleInputConfirm(scope)"
                    @blur="handleInputConfirm(scope)"
                  >
                  </el-input>
                  <el-button v-else class="button-new-tag" size="small" @click="showInput(scope)">+ New Tag</el-button>
                </template>
              </el-table-column>
              <el-table-column type="index" label="#"></el-table-column>
              <el-table-column label="姓名" prop="attr_name"></el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button type="primary" round icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)">编辑</el-button>
                  <el-button type="danger" round icon="el-icon-delete" size="mini" @click="removeParams(scope.row)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
        </el-tabs>
      </template>
    </el-card>
    <!-- 添加参数对话框 -->
    <el-dialog
      :title="'添加'+titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClose">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
            <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑参数对话框 -->
    <el-dialog
      :title="'修改'+titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClose">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
            <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      catelist: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      selectKeys: [],
      activeName: 'many',
      manyData: [],
      onlyData: [],
      addDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [{ required: true, message: '请输入属性名称', trigger: 'blur' }]
      },
      editDialogVisible: false,
      editForm: {
        attr_name: ''
      },
      editFormRules: {
        attr_name: [{ required: true, message: '请输入属性名称', trigger: 'blur' }]
      }
    }
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.catelist = res.data
    },
    handleChange () {
      if (this.selectKeys.length !== 3) {
        this.$message.error('请选择三级分类')
        this.selectKeys = []
        this.manyData = []
        this.onlyData = []
        return
      }
      this.getParamsData()
    },
    handleClick () {
      this.getParamsData()
    },
    async getParamsData () {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) return
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') this.manyData = res.data
      if (this.activeName === 'only') this.onlyData = res.data
    },
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, { attr_name: this.addForm.attr_name, attr_sel: this.activeName })
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
        this.$message.success(res.meta.msg)
        this.getParamsData()
        this.addDialogVisible = false
      })
    },
    showAddDialog () {
      this.addDialogVisible = true
    },
    addDialogClose () {
      this.$refs.addFormRef.resetFields()
    },
    showEditDialog (form) {
      this.editDialogVisible = true
      this.editForm = form
    },
    editDialogClose () {
      this.$refs.editFormRef.resetFields()
    },
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, { attr_name: this.editForm.attr_name, attr_sel: this.activeName })
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.$message.success(res.meta.msg)
        this.editDialogVisible = false
        this.getParamsData()
      })
    },
    async removeParams (form) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => {
        return err
      })
      if (confirmResult === 'cancel') return
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${form.attr_id}`)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.editDialogVisible = false
      this.getParamsData()
    },
    async handleInputConfirm (scope) {
      if (scope.row.inputValue.trim().length === 0) {
        scope.row.inputValue = ''
        scope.row.inputVisible = false
        return
      }
      scope.row.attr_vals.push(scope.row.inputValue.trim())
      scope.row.inputValue = ''
      scope.row.inputVisible = false
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${scope.row.attr_id}`, { attr_name: scope.row.attr_name, attr_sel: scope.row.attr_sel, attr_vals: scope.row.attr_vals.join(' ') })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
    },
    showInput (scope) {
      scope.row.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInputRef.focus()
      })
    },
    async handleClose (index, scope) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${scope.row.attr_id}`, { attr_name: scope.row.attr_name, attr_sel: scope.row.attr_sel, attr_vals: scope.row.attr_vals.splice(index, 1) })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
    }
  },
  computed: {
    isBtnDisabled () {
      if (this.selectKeys.length !== 3) {
        return true
      }
      return false
    },
    cateId () {
      if (this.selectKeys.length === 3) {
        return this.selectKeys[2]
      }
      return null
    },
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  },
  created () {
    this.getCateList()
  }

}
</script>

<style lang="less" scoped>
  .el-card{
    margin-top: 20px;
  }
  .el-tag{
    margin: 0 10px;
  }
  .input-new-tag{
    width: 150px;
  }
</style>
