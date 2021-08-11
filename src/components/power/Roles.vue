<template>
  <div>
    <!-- 面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色管理</el-breadcrumb-item>
    </el-breadcrumb>
       <!-- 卡片区域  -->
    <el-card>
      <!-- 添加角色按钮-->
      <el-row>
        <el-col>
          <el-button @click="addRolesDialogVisible = true" type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table
        :data="getRolesList"
        style="width: 100%"
        border stripe>
          <!--  展开二级菜单      -->
        <el-table-column type="expand">
          <template slot-scope="props">
            <el-row  v-for="(item1, i1) in props.row.children" :class="['bdottom', i1===0?'bdtop' :'','vcenter']" :key="item1.id">
              <el-col :span="6">
                <div class="grid-content bg-purple">
                  <el-tag closable @close="removeRightById(props.row,item1.id)">{{item1.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
               </div>
              </el-col>
           <!--       二级和三级权限       -->
              <el-col :span="18"><div class="grid-content bg-purple-light">
                 <el-row :class="[i2===0 ? '' :'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                   <el-col :span="6">
                       <el-tag type="success" closable @close="removeRightById(props.row,item2.id)">{{item2.authName}}</el-tag>
                       <i class="el-icon-caret-right"></i>
                   </el-col>
                   <el-col :span="18">
                     <el-tag type="warning" closable v-for="(item3) in item2.children" :key="item3.id" @close="removeRightById(props.row,item3.id)">{{item3.authName}}</el-tag>
                   </el-col>
                 </el-row>
              </div></el-col>
            </el-row>

          </template>

        </el-table-column>
          <!--  表格头部标签      -->
        <el-table-column
          label="角色名称"
          prop="roleName">
        </el-table-column>
        <el-table-column
          label="角色描述"
          prop="roleDesc">
        </el-table-column>
        <el-table-column
          label="操作">
          <template slot-scope="scope">
            <el-button round type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
            <el-button round type="danger"  size="mini" icon="el-icon-delete">删除</el-button>
            <el-button round type="warning" size="mini" icon="el-icon-setting"  @click="setshowDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
        <!-- 分配权限对话框   -->
    <el-dialog
      title="提示"
      :visible.sync="setrightsDialog"
      width="50%"
      @close="setDefkeys">
     <!-- 树形结构-->
      <el-tree
        :data="rightsList"
        show-checkbox
        node-key="id"
        :props="treeProps"
        default-expand-all
       :default-checked-keys="defKeys"
      ref="treeRef">
      </el-tree>

      <span slot="footer" class="dialog-footer">
          <el-button @click="setrightsDialog=false">取 消</el-button>
          <el-button type="primary" @click="allrights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Roles',
  data() {
    return {
      getRolesList: [],
      // 控制分配权限对话框隐藏
      setrightsDialog: false,
      rightsList: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defKeys: [],
      // 更新分配权限的id
      roleId: ''
    }
  },
  methods: {
    async getRolesData() {
      const { data: res } = await this.$http.get('roles')
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.getRolesList = res.data
      // console.log(this.getRolesList)
    },
    async removeRightById(role, roleid) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消了删除')
      }
      // console.log(confirmResult)
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${roleid}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      // 为该行数据重新渲染，不需要全部页面数据渲染
      role.children = res.data
    },
    // 分配权限对话框
    async setshowDialog(role) {
      // 将角色权限的id保存在数组中
      this.roleId = role.id
      // console.log(this.roleId)
      this.setrightsDialog = true
      // 获取权限列表
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败！')
      }
      this.rightsList = res.data
      // 调用递归获取三级权限id
      this.getLeafKeys(role, this.defKeys)
      // console.log(this.defKeys)
    },
    // 通过递归获取三级权限
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 关闭对话框清空数据id
    setDefkeys() {
      this.defKeys = []
    },
    // 点击分配权限
    allrights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // console.log(keys)
      const idStr = keys.join(',')
      // console.log(idStr)
      const { data: res } = this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('更新权限失败！')
      }
      this.getRightsList()
      this.setrightsDialog = false
    }
  },
  created() {
    this.getRolesData()
  }
}
</script>

<style lang="less" scoped>
 .el-tag{
    margin: 7px;
 }
  .bdtop{
    border-top: 1px solid #eeeeee;
  }
 .vcenter{
      display: flex;
      align-items: center;
 }
 .bdottom{
    border-bottom: 1px solid #eeeeee;
 }
</style>
