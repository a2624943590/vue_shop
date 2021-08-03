<template>
  <el-container class="home-container">
<!-- 头部区域   -->
    <el-header>
      <div>
        <img class="header-img" src="../assets/logo.png" alt="">
        <span>后台管理系统</span>
      </div>
      <el-button type="primary" @click="exitBtn">退出</el-button>
    </el-header>
    <el-container>

<!--   Aside侧边栏区域-->
      <el-aside :width="isCollpse ? '64px':'200px'">
        <div class="toggle-button" @click="toggleCollpse">|||</div>
        <el-menu
          :default-active="activePath"
          unique-opened
          class="el-menu-vertical-demo"
          background-color="#333744"
          text-color="#fff"
          active-text-color="#ffd04b"
          :collapse="isCollpse"
          :collapse-transition="false"
          router
        >
<!--          一级菜单-->
          <el-submenu :index="item.id +''" v-for="(item) in menulist" :key="item.id">
<!--            一级菜单模板-->
            <template slot="title">
<!--              图标-->
              <i :class="iconsObj[item.id]"></i>
<!--              文本-->
              <span>{{item.authName}}</span>
            </template>
<!--                二级菜单-->
              <el-menu-item @click="saveNavState('/'+subitem.path)" :index="'/'+subitem.path" v-for="subitem in item.children" :key="subitem.id">
                <template slot="title">
                  <!--              图标-->
                  <i class="el-icon-menu"></i>
                  <!--              文本-->
                  <span>{{subitem.authName}}</span>
                </template>
              </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
<!--  右侧内容区域-->
      <el-main>
<!--        路由占位符-->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      menulist: [],
      iconsObj: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollpse: false,
      activePath: ''
    }
  },
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    exitBtn () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      // console.log(this.menulist)
    },
    toggleCollpse() {
      this.isCollpse = !this.isCollpse
    },
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style  lang="less" scoped>
   .el-header{
      background-color: #373d41;
     display: flex;
     justify-content: space-between;
     padding-left: 0;
     align-items: center;
     color: #fff;
     font-size: 20px;
     > div{
       display: flex;
       align-items: center;
       img{
         margin-left: 20px;
         height: 50px;
         width: 50px;
       }
       span{
         margin-left: 20px;
       }
     }
   }
   .el-aside{
      background-color: #333744;
   }
   .el-main{
      background-color: #f6f6f6;
   }
   .home-container{
      height: 100%;
   }
   .iconfont{
      margin-right: 10px;
   }
   .el-aside .el-menu{
     border-right: none;
   }
   .toggle-button{
      background-color: #4a5064;
     font-size: 10px;
     line-height: 24px;
     color: #fff;
     text-align: center;
     letter-spacing: 0.2em;
     cursor: pointer;
   }
</style>
