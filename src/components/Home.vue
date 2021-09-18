<template>
  <el-container>
    <!-- 头部区域 -->
    <el-header>
          <div @click="returnWelcome" style="cursor: pointer">
            <img src="../assets/heima.png" alt="">
            <span>电商后台管理系统</span>
          </div>
        <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 下面主体区 -->
    <el-container>
        <!-- 侧边栏导航区域 -->
        <el-aside :width="isCollapse?'64px':'200px'">
            <div class="collapse" @click="changeCollapse">
                《||》
            </div>
            <el-menu background-color="#333744" text-color="#fff"
            :collapse="isCollapse"
            :collapse-transition="false"
            :router="true">
                <!-- 一级菜单 -->
                <!-- index要求为字符串 -->
                <el-submenu :index="item.id+''" v-for="item in menulist" :key="item.id">
                    <template slot="title">
                        <!-- 图标 -->
                        <i :class="iconObj[item.id]"></i>
                        <span>{{item.authName}}</span>
                    </template>
                    <!-- 二级菜单 -->
                    <el-menu-item-group v-for="item2 in item.children" :key="item2.id" >
                        <el-menu-item   el-menu-item :index="item2.path+''">
                            <i class="el-icon-s-grid"></i>
                            {{item2.authName}}
                        </el-menu-item>
                    </el-menu-item-group>
                </el-submenu>
            </el-menu>
        </el-aside>
        <el-main>
            <router-view></router-view>
        </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menulist: [],
      iconObj: {
        125: 'el-icon-user-solid',
        103: 'el-icon-coordinate',
        101: 'el-icon-s-goods',
        102: 'el-icon-s-order',
        145: 'el-icon-s-platform'
      },
      isCollapse: false
    }
  },
  methods: {
    logout () {
      window.sessionStorage.clear('token')
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
    },
    changeCollapse () {
      this.isCollapse = !this.isCollapse
    },
    returnWelcome () {
      this.$router.push('welcome')
    }
  },
  created () {
    this.getMenuList()
  }

}
</script>

<style lang="less" scoped>
    .el-header{
        background-color: #373d41;
        padding-left: 0;
        display: flex;
        justify-content: space-between;
        align-items: center;
        >div{
            display: flex;
            align-items: center;
            span{
                margin-left: 10px;
                color: #fff;
                font-size: 20px;
            }
        }
    }

    .el-aside{
        background-color: #333744;
        .el-menu{
            border-right: none;
        }
        .collapse{
            background-color: #4A5064;
            color: #fff;
            text-align: center;
            font-size: 20px;
            cursor: pointer;
        }
    }
    .el-main{
        background-color: #EAEDF1;
    }
    .el-container{
        height: 100%;
    }
</style>
