<template>
    <div>
          <!-- 面包屑导航 -->
          <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品列表</el-breadcrumb-item>
        </el-breadcrumb>
      <!-- 卡片试图 -->
      <el-card>
        <el-row>
            <el-col>
                <el-button type="primary" @click="show_setCateDialog">添加分类</el-button>
            </el-col>
        </el-row>
        <!-- 表格区域 -->
        <el-table :data="cateList"  border  stripe :tree-props="{children: 'children'}" row-key="cat_id" class="tree_table">
               <el-table-column label="名称" prop="cat_name"></el-table-column>
               <el-table-column label="是否有效" prop="cat_deleted">
                    <template slot-scope="scope" >
                        <i class="el-icon-success" v-if = "scope.row.cat_deleted === false" style="color: lightgreen;"></i> 
                        <i class="el-icon-error" v-else style="color: red;"></i> 
                    </template>
               </el-table-column>
                <el-table-column label="排序" prop="cat_level">
                    <template slot-scope="scope">
                        <el-tag v-if = "scope.row.cat_level === 0">一级</el-tag>
                        <el-tag type="success" v-else-if = "scope.row.cat_level === 1">二级</el-tag>
                        <el-tag type="warning" v-else>三级</el-tag>
                    </template>
                </el-table-column> -->
                <!-- 操作区域 -->
                <el-table-column label="操作">
                    <template slot-scope="scope">
                         <!-- 修改 -->
                         <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                        <!-- 删除 -->
                        <el-button type="danger" icon="el-icon-delete" size="mini" >删除</el-button>
                    </template>
                </el-table-column> -->
            </el-table>
        <!-- 分页区域 -->
        <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="pramsInfo.pagenum"
                :page-sizes="[1,2,5,10]"
                :page-size="pramsInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </el-card>
        <!-- 添加分类的对话框 -->
        <el-dialog title="添加分类" :visible.sync="setCateDialogVisible" width="50%" @close="setCateDialogVisibleClosed">
            <el-form :model="addCateForm" :rules="addCaterules" ref="addCateFormRef" label-width="100px">
                <el-form-item label="分类名称：" prop="cat_name">
                <el-input v-model="addCateForm.cat_name" ></el-input>
                </el-form-item>
                <el-form-item label="父级分类：" prop="">
                  <el-cascader v-model="selectedKeys" :options="parentList" :props="{ expandTrigger: 'hover', checkStrictly: true,
                  value:'cat_id', label:'cat_name', children:'children'}"   @change="handleChange" clearable>
                  </el-cascader>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
            <el-button @click="setCateDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default{
        data(){
           return{
            cateList:[],
            /* 查询条件 */
            pramsInfo:{
                type:3,
                pagenum:1,
                pagesize:5
            },
            /* 商品条目总数 */
            total:0,
            /* 设置添加对话框初始值 */
            setCateDialogVisible:false,
            /* 添加对话框表格 */
            addCateForm:{
                cat_name:'',
                cat_level:0,
                cat_pid:0
            },
            /* 添加对话框验证规则 */
            addCaterules:{
                cat_name:[
                { required: true, message: '请输入分类名称', trigger: 'blur' },
                { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                ]
            },
            parentList:[],
           
           
            /* 选中父级分类的Id */
            selectedKeys:[]

        }     
        },

        created(){
            this.getCateList()
        },

        methods:{
            /* 获取商品列表 */
           async getCateList(){
            const {data : res } = await this.$http.get('categories',{params:this.pramsInfo})
            if(res.meta.status !== 200 ) return this.$message.error('获取商品失败！')
            this.cateList = res.data.result
            this.total = res.data.total
            },
            /* 操作页面多少量 */
            handleSizeChange(newSize){
                this.pramsInfo.pagesize = newSize
                this.getCateList()
            },
            /* 操作页码值 */
            handleCurrentChange(newPage){
                this.pramsInfo.pagenum = newPage
                this.getCateList()
            },
            /* 显示添加对话框 */
            show_setCateDialog(){
                this.getParentList()
                this.setCateDialogVisible = true
            },
            setCateDialogVisibleClosed(){
                this.$refs.addCateFormRef.resetFields()
                this.selectedKeys = []
                this.addCateForm.cat_level = 0
                this.addCateForm.cat_pid = 0
            },
            /* 获取父级分类 */
           async getParentList(){
                const {data :res} = await this.$http.get('categories', {params:{type:'2'}})
                if(res.meta.status !== 200) return this.$message.error('获取父级分类失败！')
                this.parentList = res.data
            },
            handleChange(){

                if(this.selectedKeys.length > 0){
                   this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
                   this.addCateForm.cat_level = this.selectedKeys.length
                   return 
                }else{
                    this.addCateForm.cat_pid = 0
                    this.addCateForm.cat_level = 0
                }

            },
            /* 添加分类 */
            async addCate(){
                const {data : res} = await this.$http.post('categories',this.addCateForm)
                if(res.meta.status !== 201) return this.$message.error('添加失败！')
                this.$message.success('添加分类成功！')
                this.getCateList()
                this.setCateDialogVisible = false
            }
        }

    }
</script>

<style lang="less" scoped>
    .tree_table{
        margin-top: 15px;
    }
    .el-cascader{
        width: 100%;
    }

</style>