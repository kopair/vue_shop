<template>
    <el-container class="home-container">
      <!-- 头部 -->
      <el-header>
        <div class="header-logo">
          <img src="../assets/logo.png" alt="">
          <span>电商后台管理项目</span> 
        </div>
        <el-button type="info" @click="logout">退出</el-button>
      </el-header>
      <el-container>
        <!-- 侧边栏 -->
        <el-aside :width="isCollapse ? '64px' : '200px'">
            <div class="toggle-button" @click="toggleCollapse">|||</div>
            <el-menu
              class="el-menu-vertical-demo"
              background-color="#333744"
              text-color="#fff"
              active-text-color="#409EFF"
              unique-opened
              :collapse="isCollapse"
              :collapse-transition="false"
              router
              :default-active="activePath">
              <el-submenu  v-for="item in menuList" :key="item.id" :index="item.id+''">
                <template slot="title">
                  <i :class="iconsObj[item.id]"></i>
                  <span>{{item.authName}} </span>
                </template>
                  <el-menu-item v-for="x in item.children" :key="x.id" :index="'/'+x.path">
                  <i class="el-icon-menu"></i>
                    {{x.authName}}
                    </el-menu-item>
              </el-submenu>
              
            </el-menu>
        </el-aside>
        <!-- 主体部分 -->
        <el-main>
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
</template>

<script>
export default {
  data() {
    return {
      menuList:[],
      iconsObj :{
        '125':'iconfont icon-user',
        '103':'iconfont icon-tijikongjian',
        '101':'iconfont icon-shangpin',
        '102':'iconfont icon-danju',
        '145':'iconfont icon-baobiao'
      },
      isCollapse:true,
      // 激活链接地址
      activePath:''
    }
  },
  created() {
    this.getMenuList();
    this.activePath = this.$route.path;
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList() {
      const {data: res} = await this.$http.get('menus');
      if(res.meta.status === 200){
        this.menuList = res.data
        // console.log(this.menuList)
      }
    },
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    saveNavActive(activePath) {
      window.sessionStorage.setItem('activePath',activePath)
    }
  }
}
</script>

<style lang="less" scoped>
.home-container{
  height: 100%;
}
.el-header{
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  // padding-left: 0;
  align-items: center;
  > .header-logo{
    display: flex;
    align-items: center;
    color: white;
  }
  .header-logo img{
    width: 40px;
    height: 40px;
    margin-right: 10px
  }
}
.el-aside{
  background-color: #333744;
}
.el-main{
  background-color: #eaedf1;
}
.iconfont{
  margin-right: 10px;
}
.el-menu {
    border-right: none!important;
}
.toggle-button{
  background-color: #4A5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>