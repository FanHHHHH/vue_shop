<template>
  <div>
    <!--    面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

<!--    卡片视图区域-->
    <el-card>
<!--      警告区域-->
      <el-alert
        title="注意：只允许为第三级分类设置相关参数！"
        type="warning"
        show-icon
        :closable="false"
      >
      </el-alert>
<!--选择商品分类区域-->
    <el-row class="cat_opt">
      <el-col>
        <span>选择商品分类：</span>
<!--        选择商品分类级联选择框-->
        <el-cascader
          v-model="selectedKeys"
          :options="catList"
          :props="cascaderProps"
          @change="parentCateChange"></el-cascader>
      </el-col>
    </el-row>
<!--      Tab页签区域-->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many" :disabled="isBtnDisabled">
          <el-button type="primary" @click="addDialogVisible = true" :disabled="isBtnDisabled">添加参数</el-button>
<!--          动态参数表格-->
          <el-table :data="manyTabData" border stripe>
<!--            展开行-->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag closable v-for="(item, i) in scope.row.attr_vals" :key="i" @close="valTagClose(scope.row, i)" >{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.addInputVisible"
                  v-model="scope.row.addInputValue"
                  ref="saveTagInputRef"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showAddInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column label="#" type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="delAttr(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only" :disabled="isBtnDisabled">
          <el-button type="primary" @click="addDialogVisible = true" :disabled="isBtnDisabled">添加属性</el-button>
<!--          静态属性表格-->
          <el-table :data="onlyTabData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag closable v-for="(item, i) in scope.row.attr_vals" :key="i" @close="valTagClose(scope.row, i)">{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.addInputVisible"
                  v-model="scope.row.addInputValue"
                  ref="saveTagInputRef"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showAddInput(scope.row)">+ New Tag</el-button>
              </template>

            </el-table-column>
            <el-table-column label="#" type="index"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="delAttr(scope.row.attr_id)" >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

<!--    添加参数对话框-->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClose"
      >
      <el-form ref="addFormRef" :rules="addFormRules" :model="addForm" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addParams">确 定</el-button>
  </span>
    </el-dialog>
<!--    修改参数对话框-->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClose"
    >
      <el-form ref="editFormRef" :rules="editFormRules" :model="editForm" label-width="100px">
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
  data() {
    return {
      // 商品分类
      catList: [],
      cascaderProps: {
        checkStrictly: true,
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 级联选择框双向绑定到的数组
      selectedKeys: [],
      // 当前激活的页签名
      activeName: 'many',
      // 动态参数的数据
      manyTabData: [],
      // 静态参数的数据
      onlyTabData: [],
      // 控制添加对话框的显示与隐藏
      addDialogVisible: false,
      // 添加参数表单数据对象
      addForm: {
        attr_name: ''
      },
      // 添加参数表单数据验证对象
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入', trigger: 'blur' }
        ]
      },
      // 修改参数对话框数据
      editDialogVisible: false,
      editForm: {
        attr_name: ''
      },
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    // 获取所有商品分类列表
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('商品分类失败')
      this.catList = res.data
      // this.$message.success('获取商品列表成功')
    },
    // 级联选择框选中项发生变化出发的事件
    parentCateChange() {
      this.getParamsData()
    },
    // 处理Tab页签点击事件的处理函数
    handleTabClick() {
      // console.log((this.activeName))
      this.getParamsData()
    },
    // 获取参数列表数据
    async getParamsData() {
      // 选中的不是三级分类
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyTabData = []
        this.onlyTabData = []
        return
      }
      // 根据三级分类的ID和当前参数类型获取参数属性
      // console.log(this.selectedKeys)
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) return this.$message.error('获取失败')
      // console.log(res.data)
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 添加布尔值，控制文本框的显示与隐藏
        item.addInputVisible = false
        item.addInputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTabData = res.data
      } else {
        this.onlyTabData = res.data
      }
      // console.log(this.manyTabData)
    },
    // 监听添加对话框的关闭事件
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
      this.addForm.attr_name = ''
    },
    addParams() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 201) return this.$message.error('添加参数失败')
        this.$message.success('添加参数成功')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    // 显示修改对话框
    async showEditDialog(attrId) {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attrId}`,
        { params: { attr_sel: this.activeName } })
      if (res.meta.status !== 200) return this.$message.error('获取参数信息失败')
      this.editForm = res.data
      // console.log(this.editForm)
      this.editDialogVisible = true
    },
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
      this.editForm.attr_name = ''
    },
    editParams() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.editForm.attr_sel,
            attr_vals: this.editForm.attr_vals
          })
        if (res.meta.status !== 200) return this.$message.error('修改参数失败')
        this.editDialogVisible = false
        this.$message.success('修改参数成功')
        this.getParamsData()
      })
    },
    // 删除参数事件
    async delAttr(attrId) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        cancelButtonText: '取消',
        confirmButtonText: '确定',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 调用删除业务逻辑
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attrId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    // 展示添加输入框
    showAddInput(row) {
      row.addInputVisible = true
      // $nextTick当页面上元素重新渲染之后，才会运行回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInputRef.$refs.input.focus()
      })
    },
    // 确定添加事件
    handleInputConfirm(row) {
      // console.log(row.addInputValue)
      if (row.addInputValue.trim().length === 0) {
        row.addInputValue = ''
        row.addInputVisible = false
        return
      }
      // 如果没有return说明输入内容合法，需要做后续处理
      row.attr_vals.push(row.addInputValue.trim())
      row.addInputValue = ''
      row.addInputVisible = false
      this.saveAttrVals(row)
    },
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        })
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数项失败')
      }
      this.$message.success('修改参数项成功')
    },
    // tag关闭事件监听,删除对应的参数和选项
    valTagClose(row, idx) {
      // console.log(row.attr_vals, idx)
      row.attr_vals.splice(idx, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    // 如果按钮需要被禁用返回True 否则返回False
    isBtnDisabled() {
      return this.selectedKeys.length !== 3
    },
    // 当前选中的三级分类的ID
    cateId() {
      if (this.selectedKeys.length === 3) return this.selectedKeys[2]
      return null
    },
    // 动态计算标题的文本
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}

  .input-new-tag {
    width: 120px;
  }
</style>
