<template>
  <div>
    <!--   面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    </el-breadcrumb>
<!--    卡片视图区域-->
    <el-card>
<!--      搜索框-->
      <el-row :gutter="20">
        <el-col :span="8">
          <div>
            <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList" class="input-with-select">
              <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
            </el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>
     <!--table表格区域-->
      <el-table :data="goodslist" border stripe>
         <el-table-column type="index"></el-table-column>
         <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column width="95px" label="商品价格" prop="goods_price"></el-table-column>
        <el-table-column width="70px" label="商品重量" prop="goods_weight"></el-table-column>
        <el-table-column width="140px" label="创建时间" prop="add_time">
          <template slot-scope="scope">
            {{scope.row.add_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column width="130px" label="操作">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit"></el-button>
            <el-button size="mini" type="warning" icon="el-icon-delete" @click="removrByid(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
<!--    分页功能-->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </div>
</template>

<script>
export default {
  name: 'List',
  data() {
    return {
      goodslist: [],
      // 定义查询参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败！')
      }
      this.$message.success('获取商品列表成功！')
      // console.log(res.data);
      this.goodslist = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    async removrByid(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除!')
      }
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败！')
      }
      this.$message.success('删除参数成功！')
      this.getGoodsList()
    },
    goAddPage() {
      this.$router.push('/goods/add')
    }
  }
}
</script>

<style scoped>

</style>
