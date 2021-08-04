<template>
  <el-container class="home-container">
    <!--头部区域-->
    <el-header>
      <div>
        <img src="../assets/google1.png" alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!--页面主体区域-->
    <el-container>
      <!--侧边栏-->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <!--侧边栏菜单区-->
        <el-menu background-color="#333744" text-color="#fff"
                 active-text-color="#409BFF" unique-opened
                 :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">
          <!--一级菜单-->
          <el-submenu :index="index + ''" v-for="(item, index) in menulist" :key="item.id">
            <!--一级菜单的模板区域-->
            <template slot="title">
              <!--图标-->
              <i :class="iconsObj[item.id]"></i>
              <!--文本-->
              <span>{{item.authName}}</span>
            </template>
            <!--二级菜单-->
            <el-menu-item :index="'/' + subitem.path" v-for="subitem in item.children"
                          :key="subitem.id" @click="saveNavState('/' + subitem.path)">
              <template slot="title">
                <!--图标-->
                <i class="el-icon-menu"></i>
                <!--文本-->
                <span>{{subitem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!--右侧内容主体-->
      <el-main>
        <!--路由占位符-->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
  export default {
    name: "Home",
    data() {
      return {
        //左侧菜单数据
        menulist: [],
        iconsObj: {
          '125': 'iconfont icon-users',
          '103': 'iconfont icon-tijikongjian',
          '101': 'iconfont icon-shangpin',
          '102': 'iconfont icon-danju',
          '145': 'iconfont icon-baobiao'
        },
        // 是否折叠
        isCollapse: false,
        // 被激活的链接地址
        activePath: ''
      }
    },
    created() {
      this.getMenuList()
      this.activePath = window.sessionStorage.getItem('activePath')
    },
    methods: {
      // 退出功能
      logout() {
        // 清空token值
        window.sessionStorage.clear()
        // 跳转到login界面
        this.$router.push('/login')
      },
      //获取所有的菜单
      getMenuList() {
        this.$http.get('menus').then(res => {
          if(res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
          this.menulist = res.data.data
        })
      },
      //点击按钮，切换菜单的折叠与保存
      toggleCollapse() {
        this.isCollapse = !this.isCollapse
      },
      //保存连接的激活状态
      saveNavState(activePath) {
        window.sessionStorage.setItem('activePath', activePath)
        this.activePath = activePath
      }
    }
  }
</script>

<style lang="less" scoped>
  .home-container {
    height: 100%;
  }
  .el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;
    > div {
      display: flex;
      align-items: center;
      span {
        margin-left: 15px;
      }
      >img {
        width: 60px;
        height: 58px;
      }
    }
  }
  .el-aside {
    background-color: #333744;
    .el-menu {
      border-right: none;
    }
  }
  .el-main {
    background-color: #eaedf1;
  }
  .iconfont {
    margin-right: 10px;
  }
  .toggle-button {
    background-color: #4A5064;
    color: #fff;
    font-size: 10px;
    line-height: 24px;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
  }
</style>
