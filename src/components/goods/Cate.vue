<template>
 <div>
   <!--   面包屑导航区域-->
   <el-breadcrumb separator-class="el-icon-arrow-right">
     <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
     <el-breadcrumb-item>商品管理</el-breadcrumb-item>
     <el-breadcrumb-item>商品管理</el-breadcrumb-item>
   </el-breadcrumb>
      <!-- 卡片视图区域  -->
   <el-card>
      <el-row>
        <el-col>
          <el-button @click="showAddCat" type="primary">添加分类</el-button>
        </el-col>
      </el-row>
   <!-- 表格   -->
     <tree-table class="treeTable" border index-text="#" :show-index="true" :expand-type="false" :selection-type="false" :data = "catelist" :columns = "columns">
       <template slot="isok" slot-scope="scope">
         <i class="el-icon-success" v-if="scope.row.cat_deleted ===false" style="color: lightgreen"></i>
         <i class="el-icon-error" style="color: red" v-else></i>
       </template>
       <template slot="order" slot-scope="scope">
         <el-tag size="mini" type="primary" v-if="scope.row.cat_level ===0">一级</el-tag>
         <el-tag size="mini" type="success" v-else-if="scope.row.cat_level ===1">二级</el-tag>
         <el-tag size="mini" type="warning" v-else>三级</el-tag>
       </template>
       <template slot="opt" slot-scope="scope">
          <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditCateDialog(scope.row.cat_id)">编辑</el-button>
          <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeCateById(scope.row.cat_id)">删除</el-button>
       </template>
     </tree-table>
  <!--   分页区域  -->
     <el-pagination
       @size-change="handleSizeChange"
       @current-change="handleCurrentChange"
       :page-sizes="[3, 5, 10, 15]"
       :page-size="queryInfo.pagesize"
       layout="total, sizes, prev, pager, next, jumper"
       :total="total">
     </el-pagination>
   </el-card>

   <el-dialog
     title="添加分类"
     :visible.sync="addCatedialogVisible"
     width="50%"
     @close="addCateInfo">

     <el-form :model="addCateForm" :rules="raddCateFormRules" ref="addCateFormRef" label-width="100px" class="demo-ruleForm">
       <el-form-item label="分类名称" prop="cat_name">
         <el-input v-model="addCateForm.cat_name"></el-input>
       </el-form-item>

       <el-form-item label="父级分类">
         <el-cascader
           expand-trigger="hover"
           v-model="selectKeys"
           :options="parentCateList"
           :props="cascaderProps"
           @change="parentCateChange"
           clearable
           change-on-select>
         </el-cascader>
       </el-form-item>
     </el-form>
       <span slot="footer" class="dialog-footer">
           <el-button @click="addCatedialogVisible = false">取 消</el-button>
           <el-button type="primary" @click="addCate">确 定</el-button>
       </span>
     </el-dialog>

   <!-- 分类名称的编辑  -->
   <el-dialog
     title="编辑分类"
     :visible.sync="editCateDialogVisible"
     width="50%" @close="editCateDialogClosed">
     <!-- 添加分类的表单-->
     <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="100px">
       <el-form-item label="分类名称：" prop="cat_name">
         <el-input v-model="editCateForm.cat_name"></el-input>
       </el-form-item>
     </el-form>
     <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
   </el-dialog>
 </div>
</template>

<script>
export default {
  name: 'Cate',
  data() {
    return {
      // 商品分类的数据列表，默认为空
      catelist: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 3
      },
      total: 0,
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
        label: '是否有效',
        type: 'template',
        template: 'isok'
      }, {
        label: '排序',
        type: 'template',
        template: 'order'
      }, {
        label: '操作',
        type: 'template',
        template: 'opt'
      }],
      addCatedialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      raddCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 5 个字符', trigger: 'blur' }]
      },
      parentCateList: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectKeys: [],
      editCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ]
      },
      editCateForm: {
        cat_id: '',
        cat_name: ''
      },
      editCateDialogVisible: false
    }
  },

  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败！')
      }
      // console.log(res.data)
      this.catelist = res.data.result
      this.total = res.data.total
      //   console.log(this.catelist)
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    showAddCat() {
      this.addCatedialogVisible = true
      this.getparentCateList()
    },
    async getparentCateList() {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级列表数据失败！')
      }
      this.parentCateList = res.data
    //  console.log(res.data)
    },
    parentCateChange() {
      // console.log(this.selectKeys)
      if (this.selectKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectKeys[this.selectKeys.length - 1]
        this.addCateForm.cat_level = this.selectKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    addCateInfo() {
      this.$refs.addCateFormRef.resetFields()
      this.selectKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    addCate() {
      this.$refs.addCateFormRef.validate(async valite => {
        //     console.log(valite)
        if (!valite) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCatedialogVisible = false
        //    console.log(res)
      })
    },
    editCate() {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('categories/' + this.editCateForm.cat_id, {
          cat_name: this.editCateForm.cat_name
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改分类名称失败！')
        }
        this.$message.success('修改分类名称成功！')
        this.editCateDialogVisible = false
        this.getCateList()
      })
    },
    editCateDialogClosed() {
      this.$refs.editCateFormRef.resetFields()
    },
    async showEditCateDialog(id) {
      this.editCateDialogVisible = true
      const { data: res } = await this.$http.get('categories/' + id)
      this.editCateForm.cat_name = res.data.cat_name
      this.editCateForm.cat_id = res.data.cat_id
    },
    async removeCateById(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
        // 捕获所有报错，并直接return，下行箭头函数省略了return
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消了删除')
      }
      // 调用接口，真正发起请求
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除分类失败！')
      }
      this.$message.success('删除分类成功！')
      this.getCateList()
    }
  }
}

</script>

<style lang="less" scoped>

.treeTable{
  margin-top: 15px;
}
</style>
