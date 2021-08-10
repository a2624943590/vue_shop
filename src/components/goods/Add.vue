<template>
<div>
  <!-- 面包屑导航区域-->
  <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>添加商品</el-breadcrumb-item>
  </el-breadcrumb>
<!-- 卡片试图-->
  <el-card>
    <!-- 警告区域-->
    <el-alert
      title="添加商品信息"
      type="info" :closable="false" show-icon center>
    </el-alert>

    <!-- 步骤条区域-->
    <el-steps :active="activeIndex-0" finish-status="success" align-center>
      <el-step title="基本信息"></el-step>
      <el-step title="商品参数"></el-step>
      <el-step title="商品属性"></el-step>
      <el-step title="商品图片"></el-step>
      <el-step title="商品内容"></el-step>
      <el-step title="完成"></el-step>
    </el-steps>

    <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
    <el-tabs :tab-position="'left'" v-model="activeIndex" :before-leave="beforeTabLeave" @tab-click="tabClicked">
      <el-tab-pane label="基本信息" name="0">
        <el-form-item label="商品名称" prop="goods_name">
          <el-input v-model="addForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格" prop="goods_price">
          <el-input v-model="addForm.goods_price" type="number"></el-input>
        </el-form-item>
        <el-form-item label="商品重量" prop="goods_weight">
          <el-input v-model="addForm.goods_weight"></el-input>
        </el-form-item>
        <el-form-item label="商品数量" prop="goods_number">
          <el-input v-model="addForm.goods_number" type="number"></el-input>
        </el-form-item>
        <el-form-item label="商品分类" prop="goods_cat">
          <el-cascader
            v-model="addForm.goods_cat"
            :options="catelist"
            :props="cateProps"
            @change="handleChange"></el-cascader>
        </el-form-item>
      </el-tab-pane>
      <el-tab-pane label="商品参数" name="1">
        <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
          <!-- 复选框组-->
          <el-checkbox-group v-model="item.attr_vals">
            <el-checkbox v-for="(cb,i) in item.attr_vals" :key="i" :label="cb" border></el-checkbox>
          </el-checkbox-group>
        </el-form-item>
      </el-tab-pane>
      <el-tab-pane label="商品属性" name="2">
        <!-- 渲染表单的item项-->
        <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
            <el-input v-model="item.attr_vals"></el-input>
        </el-form-item>
      </el-tab-pane>
      <el-tab-pane label="商品图片" name="3">
        <el-upload
          :action="baseURL"
          :on-preview="handlePreview"
          :on-remove="handleRemove"
          list-type="picture"
        :headers="headerObj"
        :on-success="handleSuccess">
          <el-button size="small" type="primary">点击上传</el-button>
          <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
        </el-upload>
      </el-tab-pane>
      <el-tab-pane label="商品内容" name="4">
        <!-- 富文本编辑器组件-->
        <quill-editor v-model="addForm.goods_introduce"></quill-editor>
        <!-- 添加商品的按钮-->
        <el-button type="primary" class="btnAdd" @click="add">添加商品</el-button>
      </el-tab-pane>
     </el-tabs>
    </el-form>
  </el-card>
<!--图片预览对话框-->
  <el-dialog
    title="图片预览"
    :visible.sync="previewVisible"
    width="50%">
  <img :src="previewPath" class="previewImg">
  </el-dialog>
</div>
</template>

<script>
import _ from 'lodash'
export default {
  name: 'Add',
  data() {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        // 商品所属的分类数组
        goods_cat: [],
        // 上传的图片
        pics: [],
        // 商品内容的详情描述
        goods_introduce: '',
        // 商品的参数（数组），包含 `动态参数` 和 `静态属性`
        attrs: []
      },
      catelist: [],
      addFormRules: {
        goods_name: [{ required: true, message: '请输入商品名称', trigger: 'blur' }],
        goods_price: [{ required: true, message: '请输入商品价格', trigger: 'blur' }],
        goods_weight: [{ required: true, message: '请输入商品重量', trigger: 'blur' }],
        goods_number: [{ required: true, message: '请输入商品数量', trigger: 'blur' }],
        goods_cat: [{ required: true, message: '请选择商品分类', trigger: 'blur' }]
      },
      cateProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      manyTableData: [],
      onlyTableData: [],
      baseURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 图片上传请求头
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewPath: '',
      previewVisible: false
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败！')
      }
      this.catelist = res.data
      console.log(res.data)
    },
    handleChange() {
      // console.log(this.addForm.goods_cat);
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    // 切换标签页触发的函数
    beforeTabLeave(activeName, oldActiveName) {
      // return false会组织标签页的切换
      // 不可以直接在箭头函数中return false
      let flag = true
      this.$refs.addFormRef.validate(valid => {
        // eslint-disable-next-line no-return-assign
        return flag = valid
      })
      if (!flag || (oldActiveName === '0' && this.addForm.goods_cat.length !== 3)) {
        this.$message.error('请填写选择所有必须项！')
        return false
      }
    },
    async tabClicked() {
      // 访问动态参数面板
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: { sel: 'many' }
        })
        if (res.meta.status !== 200) {
          return this.$message.error('获取动态参数列表失败！')
        }
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: { sel: 'only' }
        })
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品属性列表失败！')
        }
        this.onlyTableData = res.data
      }
    },
    // 处理图片预览效果
    handlePreview(file) {
      console.log(file)
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    handleRemove(file) {
      // console.log(file)
      const filePath = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex(x => x.pic === filePath)
      this.addForm.pics.splice(i, 1)
    },
    // 监听图片上传成功事件
    handleSuccess(response) {
      const picInfo = {
        pic: response.data.tmp_path
      }
      this.addForm.pics.push(picInfo)
    },
    add() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项！')
        }
        // 执行修改的业务逻辑
        // 使用Lodash做深拷贝
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // console.log(form)
        // 处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          // console.log(newInfo);
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })

        form.attrs = this.addForm.attrs
        // console.log(form);
        // 发起请求修改商品
        // 商品的名称必须唯一
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) {
          this.$message.error('添加商品失败！')
        }
        this.$message.success('添加商品成功！')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox{
  margin: 0 10px 0 0 !important;
}
.previewImg{
  width: 100%;
}
.btnAdd{
  margin-top: 15px;
}
</style>
