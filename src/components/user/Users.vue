<template>
 <div>
       <!--   面包屑导航区域-->
   <el-breadcrumb separator="/">
     <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
     <el-breadcrumb-item><a href="/">用户管理</a></el-breadcrumb-item>
     <el-breadcrumb-item>用户列表</el-breadcrumb-item>
   </el-breadcrumb>

         <!--   卡片视图-->
   <el-card class="box-card">
     <el-row :gutter="20">
       <el-col :span="8">
         <div>
           <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList" class="input-with-select">
             <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
           </el-input>
         </div>
       </el-col>
       <el-col :span="4">
         <el-button type="primary" @click="dialogVisible = true">添加用户</el-button>
       </el-col>
     </el-row>
        <!--     用户列表区域-->
     <el-table :data="userlist" border stripe>
       <el-table-column type="index" label="#"></el-table-column>
       <el-table-column label="姓名" prop="username"></el-table-column>
       <el-table-column label="邮箱" prop="email"></el-table-column>
       <el-table-column label="电话" prop="mobile"></el-table-column>
       <el-table-column label="角色" prop="role_name"></el-table-column>
       <el-table-column label="状态" >
         <template slot-scope="scope">
           <el-switch v-model="scope.row.mg_state" @change="userInfoState(scope.row)" ></el-switch>
         </template>
       </el-table-column>
       <el-table-column label="操作"  width="180px">
         <template slot-scope="scope">
           <el-button round type="primary" size="mini" @click="editUserInfo(scope.row.id)" icon="el-icon-edit"></el-button>
           <el-button round type="danger" @click="removeUserInfo(scope.row.id)" size="mini" icon="el-icon-delete"></el-button>

           <el-tooltip content="分配角色" placement="top" :enterable="false">
             <el-button round type="warning" size="mini" icon="el-icon-setting" @click="setRoaleRights(scope.row)"></el-button>
           </el-tooltip>
         </template>
       </el-table-column>
     </el-table>
          <!--  分页区域   -->
     <el-pagination
       @size-change="handleSizeChange"
       @current-change="handleCurrentChange"
       :current-page="queryInfo.pagenum"
       :page-sizes="[1, 2, 5, 10]"
       :page-size="queryInfo.pagesize"
       layout="total, sizes, prev, pager, next, jumper"
       :total="total">
     </el-pagination>
   </el-card>
<!--   //这是弹出层对话框 添加用户-->
   <el-dialog
     title="添加用户"
     :visible.sync="dialogVisible"
     width="50%"
     @close="addDialogClosed">

     <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="80px" status-icon>
       <el-form-item label="用户名" prop="username">
         <el-input v-model="addForm.username"></el-input>
       </el-form-item>
       <el-form-item label="密码" prop="password">
         <el-input v-model="addForm.password" type="password"></el-input>
       </el-form-item>
       <el-form-item label="确认密码" prop="checkpass">
         <el-input v-model="addForm.checkpass" type="password"></el-input>
       </el-form-item>
       <el-form-item label="邮箱" prop="email">
         <el-input v-model="addForm.email"></el-input>
       </el-form-item>
       <el-form-item label="手机" prop="mobile">
         <el-input v-model="addForm.mobile"></el-input>
       </el-form-item>
     </el-form>

     <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="adduser">确 定</el-button>
      </span>
   </el-dialog>

         <!--只是修改用户信息的对话框   -->
   <el-dialog
     title="修改用户信息"
     :visible.sync="eidtdialogVisible"
     width="40%"
   @close="editFormRet">

     <el-form ref="editForm" :rules="editFormRules" :model="editForm" label-width="80px">
       <el-form-item label="用户名">
         <el-input v-model="editForm.username" disabled></el-input>
       </el-form-item>
       <el-form-item label="邮箱" prop="email">
         <el-input v-model="editForm.email"></el-input>
       </el-form-item>
       <el-form-item label="电话号码" prop="mobile">
         <el-input v-model="editForm.mobile"></el-input>
       </el-form-item>
     </el-form>
     <span slot="footer" class="dialog-footer">
        <el-button @click="eidtdialogVisible=false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
     </span>
   </el-dialog>
                <!--这是分配角色的对话框-->
   <el-dialog
     title="提示"
     :visible.sync="editUsersRoles"
     width="50%"
    @close="setRolesDel">
     <p>当前的用户：{{userInfo.username}}</p>
     <p>当前的角色：{{userInfo.role_name}}</p>
     <p>分配新角色：
       <el-select v-model="selectRoleId" placeholder="请选择">
         <el-option
           v-for="item in roleslist"
           :key="item.id"
           :label="item.roleName"
           :value="item.id">
         </el-option>
       </el-select>
     </p>
     <span slot="footer" class="dialog-footer">
    <el-button @click="editUsersRoles = false">取 消</el-button>
    <el-button type="primary" @click="saveRolesInfo">确 定</el-button>
  </span>
   </el-dialog>
 </div>
</template>

<script>
export default {
  name: 'Users',
  data() {
    var validatorPass = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.addForm.password) {
        callback(new Error('两次输入密码不一致!'))
        //  console.log(typeof (value))
        // console.log(this.addForm.password)
        // console.log(this.addForm.checkpass)
      } else {
        return callback()
      }
    }
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
      if (regEmail.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的邮箱！'))
    }
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^(13[0-9]|14[01456879]|15[0-35-9]|16[2567]|17[0-8]|18[0-9]|19[0-35-9])\d{8}$/
      if (regMobile.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的手机号！'))
      // console.log(value)
    }
    return {
      queryInfo: {
        query: '',
        // 当前的页数
        pagenum: 1,
        pagesize: 5
      },
      userlist: [],
      total: 0,
      // 控制添加用户对话框的隐藏
      dialogVisible: false,
      // 修改用户对话框
      eidtdialogVisible: false,
      checkpass: '',
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: '',
        checkpass: ''
      },
      editForm: {},
      addFormRules: {
        username: [
          { required: true, message: '请输入登录用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 6 到 10 个字符', trigger: 'blur' }
        ],
        checkpass: [
          { validator: validatorPass, trigger: 'blur' }
        ],
        email: [
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 定义修改用户信息验证规则
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 这是分配角色对话框的隐藏
      editUsersRoles: false,
      // 需要分配权限的用户信息
      userInfo: {},
      // 所有角色的列表
      roleslist: [],
      // 已选中的id值
      selectRoleId: ''
    }
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error('获取用户列表失败')
      this.userlist = res.data.users
      this.total = res.data.total
      // console.log(res)
      // console.log(this.userlist)
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    async userInfoState(ThiState) {
      // console.log(ThiState)
      const { data: res } = await this.$http.put(`users/${ThiState.id}/state/${ThiState.mg_state}`)
      console.log(res)
      if (res.meta.status !== 200) {
        ThiState.mg_state = !ThiState.mg_state
        return this.$message.error('用户状态更新失败！')
      }
      this.$message.success('更新用户状态成功！')
    },
    // 监听添加用户对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 监听修改用户信息对话框的关闭事件
    editFormRet() {
      this.$refs.editForm.resetFields()
    },
    // 点击按钮添加用户
    adduser() {
      this.$refs.addFormRef.validate(async valited => {
        // console.log(valited)
        if (!valited) return
        const { data: res } = await this.$http.post('users', {
          username: this.addForm.username,
          password: this.addForm.password,
          email: this.addForm.email,
          mobile: this.addForm.mobile
        })
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败！')
        }
        this.$message.success('添加用户成功！')
        this.dialogVisible = false
        // 重新刷新用户的列表
        this.getUserList()
      })
    },
    // 查询修改用户的信息
    async editUserInfo(id) {
      this.eidtdialogVisible = true
      // console.log(id)
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败！')
      }
      this.editForm = res.data
      // console.log(res)
      // console.log(res.data.mobile)
    },
    // 更新用户的信息
    editUser() {
      // 关闭对话款
      this.eidtdialogVisible = false
      this.$refs.editForm.validate(async valid => {
        if (!valid) return
        console.log(valid)
        console.log(this.editForm.id)
        const { data: res } = await this.$http.put('users/' + this.editForm.id, {
          mobile: this.editForm.mobile,
          email: this.editForm.email
        })
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('更新用户失败！')
        }
        this.$message.success('更新用户成功！')
        // 刷新用户列表
        this.getUserList()
      })
    },
    // 删除用户信息操作
    async removeUserInfo(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      //  console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消了删除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) {
        this.$message.error('删除失败！')
      }
      this.$message.success('删除成功！')
      this.getUserList()
    },
    // 分配角色的点击事件
    async setRoaleRights(userInfo) {
      this.editUsersRoles = true
      this.userInfo = userInfo
      // console.log(userInfo)

      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.roleslist = res.data
    },

    // 点击提交分配角色
    async saveRolesInfo() {
      this.editUsersRoles = false
      if (!this.selectRoleId) {
        return this.$message.error('请选择分配的角色')
      }
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, {
        rid: this.selectRoleId
      })
      if (res.meta.status !== 200) {
        return this.$message.error('更新角色失败！')
      }
      this.getUserList()
      this.$message.success('更新角色成功！')
    },
    setRolesDel() {
      this.selectRoleId = ''
      this.userInfo = {}
    }
  },
  created() {
    this.getUserList()
  }
}
</script>

<style lang="less" scoped>

</style>
