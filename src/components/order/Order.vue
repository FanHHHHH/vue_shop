<template>
  <div>
    <!--    面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table :data="orderList" style="width: 100%" border stripe>
        <el-table-column label="#" type="index"> </el-table-column>
        <el-table-column prop="order_number" label="订单编号" width="600">
        </el-table-column>
        <el-table-column prop="order_price" label="订单价格"> </el-table-column>
        <el-table-column prop="pay_status" label="是否付款">
          <template slot-scope="scope">
            <el-tag type="danger" v-if="scope.row.pay_status === '0'"
              >未付款</el-tag
            >
            <el-tag type="success" v-else>已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货"> </el-table-column>
        <el-table-column prop="create_time" label="下单时间" width="200px">
          <template slot-scope="scope">
            {{ scope.row.create_time | dataFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180">
          <template slot-scope="scope">
            <el-tooltip
              class="item"
              effect="light"
              content="修改地址"
              placement="top-end"
            >
              <el-button
                type="primary"
                icon="el-icon-edit"
                size="mini"
                @click="showEditDialog(scope.row)"
              ></el-button>
            </el-tooltip>
            <el-tooltip
              class="item"
              effect="light"
              content="物流信息"
              placement="top-end"
            >
              <el-button
                type="success"
                icon="el-icon-location"
                size="mini"
                @click="showProgressBox"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <el-dialog
      title="提示"
      :visible.sync="editDialogVis"
      width="50%"
      @close="editDialogClose"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
      >
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader
            :options="cityData"
            v-model="editForm.address1"
            label-width="100px"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="editForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVis = false">取 消</el-button>
        <el-button type="primary" @click="editDialogVis = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <el-dialog
      title="物流进度"
      :visible.sync="progressDialogVis"
      width="50%"
    >
      <span>这是一段信息</span>
       <span slot="footer" class="dialog-footer">
        <el-button @click="progressDialogVis = false">取 消</el-button>
        <el-button type="primary" @click="progressDialogVis = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'

export default {
  data() {
    return {
      orderList: [],
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 3,
        user_id: '',
        pay_status: '',
        is_send: '',
        order_fapiao_title: [],
        order_fapiao_company: '',
        order_fapiao_content: '',
        consignee_addr: ''
      },
      total: 3,
      editDialogVis: false,
      editForm: {
        address1: [],
        address2: ''
      },
      editFormRules: {
        address1: [
          { required: true, message: '请选择省市区/县地址', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      addressFrom: [],
      progressDialogVis: false,
      progressInfo: []
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('请求订单列表失败')
      }
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(size) {
      this.queryInfo.pagesize = size
      this.getOrderList()
    },
    handleCurrentChange(curPage) {
      this.queryInfo.pagenum = curPage
      this.getOrderList()
    },
    showEditDialog(row) {
      this.editDialogVis = true
      // console.log(row)
    },
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },
    showProgressBox() {
      this.progressDialogVis = true
      // const { data: res } = await this.$http.get('/kuaidi/804909574412544580')
      // if (res.meta.status !== 200) {
      //   return this.$message.error('获取物流信息失败')
      // }
      // this.progressInfo = res.data
      // console.log(this.progressInfo)
    },
    progressDialogClose() {}
  },
  computed: {}
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
