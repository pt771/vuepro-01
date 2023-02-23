<template>
        <el-container class="form-container">
        <!-- 头部 -->
        <el-header>
            <div>
                <img src="../assets/头像.jpg" alt="">
                <span>管理后台</span>
            </div>
            <el-button type="info" @click="logout">退出</el-button>
        </el-header> 
        <!-- 页面布局 -->
        <el-container>
        <!-- 左侧边栏 -->
        <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleChange">|||</div>
        <el-menu background-color="#333744" text-color="#fff" 
        active-text-color="#409EFF" unique-opened 
        :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath"
        >
        <!-- 一级菜单 -->
        <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
        <template slot="title">
          <i :class="iconObj[item.id]"></i>
          <span>{{ item.authName }}</span>
        </template>
        <!-- 二级菜单 -->
        <el-menu-item :index="'/' + subitem.path" v-for="subitem in item.children" 
            :key="subitem.id" @click="saveNavLink('/' + subitem.path)"
        >
         <template slot="title">
          <i class="el-icon-menu"></i>
          <span>{{ subitem.authName }}</span>
        </template>
        </el-menu-item>
        </el-submenu>
    </el-menu>
        </el-aside>
        <!-- 右侧内容 -->
        <el-main>
            <router-view></router-view>
        </el-main>
        </el-container>
        </el-container>
</template>

<script>
    export default{
        data(){
            return{
                menuList:[],
                iconObj:{
                    '125' :'el-icon-s-custom',
                    '103' :'el-icon-coin',
                    '101' :'el-icon-shopping-bag-1',
                    '102' :'el-icon-notebook-2',
                    '145' :'el-icon-data-analysis',
                },
                isCollapse:false,
                activePath:[]
            }
        },
        created(){
            this.getMenuList()
            this.activePath = window.sessionStorage.getItem('activePath')
        },
        methods:{
            logout(){
                window.sessionStorage.clear()
                this.$router.push('/login')
            },
            async getMenuList(){
            const {data:res} = await this.$http.get('menus')
            if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
            this.menuList = res.data 
            },
            toggleChange(){
                this.isCollapse = !this.isCollapse
            },
            saveNavLink(activePath){
                window.sessionStorage.setItem('activePath', activePath)
                this.activePath = activePath
            }
        }
    }   
</script>

<style lang="less" scoped>
.form-container{
    height: 100%;
}
.el-header{
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;
        >div{
            display: flex;
            align-items: center;
            span{
                margin-left: 15px;
            }
        }
    img{
        height: 55px;
        width: 55px;
        border-radius: 50%;
    }
}
.el-aside{
    background-color: #333744;
}
.el-main{
    background-color: #EAEDF1;
}
.el-menu{
    border-right: none;
    span{
        margin-left: 10px;
    }
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