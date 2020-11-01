<template>
  <div>
    <!--    面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 提示消息 -->
      <el-alert
        title="添加商品信息"
        type="info"
        :closable="false"
        center
        show-icon
      >
      </el-alert>
      <!-- 步骤条区域 -->
      <el-steps
        :active="activeTabName - 0"
        finish-status="success"
        align-center
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- 表单区域 -->
      <el-form
        :model="addGoodsForm"
        :rules="addGoodsFormRules"
        ref="addGoodsFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-tabs
          v-model="activeTabName"
          tab-position="left"
          :before-leave="beforeTabLeave"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addGoodsForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input
                v-model="addGoodsForm.goods_price"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input
                v-model="addGoodsForm.goods_weight"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input
                v-model="addGoodsForm.goods_number"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addGoodsForm.goods_cat"
                :options="catList"
                :props="cascaderProps"
                @change="handleCascaderChange"
                clearable
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item
              v-for="item in manyTableData"
              :key="item.attr_id"
              :label="item.attr_name"
            >
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  v-for="(s, idx) in item.attr_vals"
                  :label="s"
                  :key="idx"
                  border
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              v-for="item in onlyTableData"
              :key="item.attr_id"
              :label="item.attr_name"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload
              :action="uploadURL"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headerObj"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor
              ref="myQuillEditor"
              v-model="addGoodsForm.goods_introduce"
            />
            <el-button class="btnAdd" type="primary" @click="add">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <el-dialog :visible.sync="picPreviewdialogVis" width="50%" title="图片预览">
      <img :src="previewPath" alt="图片" class="previewImg" />
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  data() {
    return {
      //   当前激活的标签名字
      activeTabName: '0',
      //   添加商品表单数据对象
      addGoodsForm: {
        goods_name: '',
        goods_price: '0',
        goods_weight: '0',
        goods_number: '0',
        goods_cat: '',
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      // 添加商品表单验证规则
      addGoodsFormRules: {
        goods_name: [
          {
            required: true,
            message: '请输入商品名称',
            triggr: 'blur'
          }
        ],
        goods_price: [
          {
            required: true,
            message: '请输入商品价格',
            triggr: 'blur'
          }
        ],
        goods_weight: [
          {
            required: true,
            message: '请输入商品重量',
            triggr: 'blur'
          }
        ],
        goods_number: [
          {
            required: true,
            message: '请输入商品数量',
            triggr: 'blur'
          }
        ],
        goods_cat: [
          {
            type: 'array',
            required: true,
            message: '请选择商品分类',
            triggr: 'blur'
          }
        ]
      },
      // 商品分类列表
      catList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 动态参数列表
      manyTableData: [],
      // 静态属性列表数据
      onlyTableData: [],
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 图片上传组件的headers
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewPath: '',
      picPreviewdialogVis: false
    }
  },
  created() {
    this.getCatList()
  },
  methods: {
    //   监听标签被点击事件
    handleTabClick(tab, event) {
      //   this.active = Number(tab.name)
      //   console.log(tab, event)
    },
    async getCatList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败')
      }
      this.catList = res.data
      // console.log(this.catList)
    },
    handleCascaderChange(key) {
      if (this.addGoodsForm.goods_cat.length !== 3) {
        this.addGoodsForm.goods_cat.length = []
      }
    },
    beforeTabLeave(activeName, oldActiveName) {
      // console.log(activeName, oldActiveName)
      if (oldActiveName === '0' && this.addGoodsForm.goods_cat.length !== 3) {
        this.$message.error('请选择商品分类')
        return false
      }
    },
    async tabClicked() {
      if (this.activeTabName === '1') {
        const { data: res } = await this.$http.get(
          `categories/${this.catId}/attributes`,
          {
            params: { sel: 'many' }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('请求动态参数列表失败')
        }
        res.data.forEach((item) => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
        // console.log(this.manyTableData)
      } else if (this.activeTabName === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.catId}/attributes`,
          {
            params: { sel: 'only' }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品属性失败')
        }
        this.onlyTableData = res.data
      }
    },
    handlePreview(file) {
      this.previewPath = file.response.data.url
      this.picPreviewdialogVis = true
    },
    handleRemove(file) {
      const filePath = file.response.data.tmp_path
      const idx = this.addGoodsForm.pics.findIndex((x) => {
        return x.pic === filePath
      })
      this.addGoodsForm.pics.splice(idx, 1)
      console.log(this.addGoodsForm)
    },
    handleSuccess(response) {
      if (response.meta.status !== 200) {
        return this.$message.error('图片上传失败')
      }
      this.addGoodsForm.pics.push({ pic: `/${response.data.tmp_path}` })
      // console.log(this.addGoodsForm)
    },
    add() {
      this.$refs.addGoodsFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项目')
        }
        // 处理goods_cat，lodash深拷贝
        const form = _.cloneDeep(this.addGoodsForm)
        form.goods_cat = this.addGoodsForm.goods_cat.join(',')
        // 处理动态参数和静态属性
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join('')
          }
          this.addGoodsForm.attrs.push(newInfo)
        })
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addGoodsForm.attrs.push(newInfo)
        })
        form.attrs = this.addGoodsForm.attrs
        // console.log(form)
        // 发起提交请求，添加商品
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) return this.$message.error('创建商品失败')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    catId() {
      if (this.addGoodsForm.goods_cat.length === 3) {
        return this.addGoodsForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-steps {
  margin: 15px 0;
}
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.previewImg {
  width: 100%;
}
.btnAdd {
  margin-top: 15px;
}
</style>
