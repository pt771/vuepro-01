<template>
    <div>
         <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <!-- 警告区域 -->
            <el-alert
                title="注意：只允许为第三级分类设置相关参数！"
                type="warning"
                :closable = "false"
                show-icon>
            </el-alert>
            <!-- 选择区域 -->
            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类：</span>
                    <!-- 级联选择框 -->
                    <el-cascader
                        v-model="selectedCateKeys"
                        :options="cateList"
                        :props="{ expandTrigger:'hover',  value:'cat_id',label:'cat_name',children:'children'  }"
                        @change="handleChange"></el-cascader>
                </el-col>
            </el-row>
            <!-- tabs页签 -->
            <el-tabs v-model="activeName" @tab-click="handleClick">
                <el-tab-pane label="动态参数" name="many">
                    <el-button type="primary" size="mini" :disabled="isBtnDisable">添加参数</el-button>
                </el-tab-pane>
                <el-tab-pane label="静态属性" name="only">
                    <el-button type="primary" size="mini" :disabled="isBtnDisable">添加属性</el-button>
                </el-tab-pane>
            </el-tabs>
        </el-card>
    </div>
</template>

<script>
    export default{
       data(){
          return{
            cateList:[],
            selectedCateKeys:[],
            activeName:'many',
           
          }
        }, 

        created(){
            this.getCateList()
        },

        computed:{
            isBtnDisable(){
                if(this.selectedCateKeys.length !== 3){
                    return true
                }
                return false
            },
            cateId(){
                if(this.selectedCateKeys.length === 3){
                    return this.selectedCateKeys[2]
                }
                return null
            }
        },

        methods:{
            async  getCateList(){
            const {data:res} = await this.$http.get('categories')
            
            if(res.meta.status !== 200) return this.$message.error('获取商品列表失败！')
           
            this.cateList = res.data
            },
            handleClick(){ this.getCateParams()},
            handleChange(){
                this.getCateParams()
            },

            async getCateParams(){
                if(this.selectedCateKeys.length !== 3) {
                this.selectedCateKeys = []
                return
               }
               const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {params: { sel: this.activeName }})
                if (res.meta.status !== 200) {return this.$message.error('获取参数列表失败!') }
                console.log(res);
            }

        }

    }
</script>

<style lang="less" scoped>
    .cat_opt{
        margin: 15px 0 ;
    }
</style>