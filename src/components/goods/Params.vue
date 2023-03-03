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
                    <el-cascader v-model="selectedCateKeys" :options="cateList" :props="{ expandTrigger:'hover', ...cateProps  }"
                        @change="handleChange">
                    </el-cascader>
                </el-col>
            </el-row>
            <!-- tabs页签 -->
            <el-tabs v-model="activeName" @tab-click="handleClick">
                <el-tab-pane label="动态参数" name="many">
                    <el-button type="primary" size="mini" :disabled="isBtnDisable" @click="setParamsDialogVisible = true">添加参数</el-button>
                        <el-table :data="manyProps" border stripe>
                            <el-table-column type="expand" >
                                <template slot-scope="scope">
                                    <!-- 添加的标签 -->
                                    <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="deleteNewTag(i,scope.row)">
                                        {{ item }}
                                    </el-tag>
                                    <!-- 输入文本框 -->
                                    <el-input
                                        class="input-new-tag"
                                        v-if="scope.row.inputVisible"
                                        v-model="scope.row.inputValue"
                                        ref="saveTagInput"
                                        size="small"
                                        @keyup.enter.native="handleInputConfirm(scope.row)"
                                        @blur="handleInputConfirm(scope.row)">
                                    </el-input>
                                    <!-- 文本框点击按钮 -->
                                    <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                                </template>
                            </el-table-column>
                            <el-table-column type="index" ></el-table-column>
                            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                            <el-table-column label="操作" >
                                <template slot-scope="scope">
                                    <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                    <el-button type="danger" size="mini" icon="el-icon-delete" @click="deleteParams(scope.row.attr_id)">删除</el-button>
                                </template>
                            </el-table-column>
                        </el-table>
                </el-tab-pane>
                <el-tab-pane label="静态属性" name="only">
                    <el-button type="primary" size="mini" :disabled="isBtnDisable" @click="setParamsDialogVisible = true">添加属性</el-button>
                    <el-table :data="onlyProps" border stripe>
                        <el-table-column type="expand" >
                                <template slot-scope="scope">
                                    <!-- 添加的标签 -->
                                    <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="deleteNewTag(i,scope.row)">
                                        {{ item }}
                                    </el-tag>
                                    <!-- 输入文本框 -->
                                    <el-input
                                        class="input-new-tag"
                                        v-if="scope.row.inputVisible"
                                        v-model="scope.row.inputValue"
                                        ref="saveTagInput"
                                        size="small"
                                        @keyup.enter.native="handleInputConfirm(scope.row)"
                                        @blur="handleInputConfirm(scope.row)">
                                    </el-input>
                                    <!-- 文本框点击按钮 -->
                                    <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                                </template>
                            </el-table-column>
                            <el-table-column type="index" ></el-table-column>
                            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                            <el-table-column label="操作" >
                                <template slot-scope="scope">
                                    <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                    <el-button type="danger" size="mini" icon="el-icon-delete" @click="deleteParams(scope.row.attr_id)">删除</el-button>
                                </template>
                            </el-table-column>
                        </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>
       <!--  添加参数对话框 -->
       <el-dialog :title="'添加' + titleText" :visible.sync="setParamsDialogVisible" width="50%"  @close="setParamsDialogClosed">
        <el-form :model="addParamsForm" :rules="paramsRules" ref="paramsRuleRef" label-width="100px" class="demo-ruleForm">
            <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="addParamsForm.attr_name"></el-input>
            </el-form-item>
        </el-form>
            <span slot="footer" class="dialog-footer">
            <el-button @click="setParamsDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addParams">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 编辑对话框 -->
        <el-dialog :title="'修改' + titleText" :visible.sync="seteditDialogVisible" width="50%"  @close="seteditDialogClosed">
        <el-form :model="addeditForm" :rules="editRules" ref="editRuleRef" label-width="100px" class="demo-ruleForm">
            <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="addeditForm.attr_name"></el-input>
            </el-form-item>
        </el-form>
            <span slot="footer" class="dialog-footer">
            <el-button @click="seteditDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addEdit">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default{
       data(){
          return{
            cateList:[],
            selectedCateKeys:[],
            activeName:'many',
            cateProps:{
                value:'cat_id',
                label:'cat_name',
                children:'children'
            },
            manyProps:[],
            onlyProps:[],
            paramsList:[],
            setParamsDialogVisible:false,
            /* 参数数据对象 */
            addParamsForm:{},
            paramsRules:{
                attr_name:[
                { required: true, message: '请输入参数名称', trigger: 'blur' },

                ]
            },
            /* 修改对话框 */
            seteditDialogVisible : false,
            addeditForm:{},
            editRules:{
                attr_name:[
                { required: true, message: '请输入参数名称', trigger: 'blur' },
                ]
            },
            /* 控制标签的数据对象 */
            inputVisible:false,
            inputValue:'' 
          }
        }, 

        created(){
            this.getCateList()
        },

        methods:{
            async  getCateList(){
            const {data:res} = await this.$http.get('categories')
            
            if(res.meta.status !== 200) return this.$message.error('获取商品列表失败！')
           
            this.cateList = res.data
            },
            handleClick(){ 
                this.getCateParams()
                
            },
            handleChange(){
                this.getCateParams()

            },

            async getCateParams(){
                if(this.selectedCateKeys.length !== 3) {
                this.selectedCateKeys = []
                this.manyProps = []
                this.onlyProps = []
                return
               }
               const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {params:{sel:this.activeName}})
                if (res.meta.status !== 200) {return this.$message.error(res.meta.msg) }

               res.data.forEach(item => {
                    item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
                    item.inputValue = ''
                    item.inputVisible = false
               });

               
                if(this.activeName === 'many'){
                    this.manyProps = res.data
                }else{
                    this.onlyProps = res.data
                }
            },
            setParamsDialogClosed(){
                this.$refs.paramsRuleRef.resetFields()
            },
            addParams(){
                this.$refs.paramsRuleRef.validate(async valid =>{
                    if(!valid)  return
                   const {data:res} = await this.$http.post(`categories/${this.cateId}/attributes`,{
                        attr_name:this.addParamsForm.attr_name,
                        attr_sel:this.activeName
                    })
                    if(res.meta.status !== 201) return this.$message.error(res.meta.msg)
                    this.$message.success(res.meta.msg)
                    this.paramsList = res.data
                    this.setParamsDialogVisible = false
                    this.getCateParams()
                })
            },
            /* 根据id获取信息 */
         async showEditDialog(attr_id){
                const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`,{params:{
                     attr_sel:this.activeName
                }})
                if(res.meta.status !== 200) return this.$message.error('查询失败！')
                this.addeditForm = res.data
                this.seteditDialogVisible = true
            },
            seteditDialogClosed(){
                this.$refs.editRuleRef.resetFields()
            },
           addEdit(){
                this.$refs.editRuleRef.validate(async valid =>{
                    if(!valid)  return
                   const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.addeditForm.attr_id}`,{
                        attr_name:this.addeditForm.attr_name,
                        attr_sel:this.activeName
                    })
                    if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
                    this.$message.success('添加成功！')
                    this.seteditDialogVisible = false
                    this.getCateParams()
                })
            },
            async deleteParams(attr_id){
                const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', 
                {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
                }).catch(err => err);
                if(confirmResult !== 'confirm') {
                    return this.$message.info('已取消删除！')
                }
                const {data:res} =await  this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
                if(res.meta.status !== 200) return this.$message.error('删除失败！')
                this.$message.success('删除成功！')
                this.getCateParams()
            },
            /* 文本框的控制方法 */
            handleInputConfirm(row){
                if(row.inputValue.trim().length === 0 ){
                    row.inputValue = ''
                    row.inputVisible = false
                    return
                }
                row.attr_vals.push(row.inputValue.trim())   
                row.inputValue = ''
                row.inputVisible = false
                /* 保存修改的参数 */
                this.saveTagParams(row)
            },
            /* 抽离的保存请求接口 */
            async saveTagParams(row){
                const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,{
                    attr_name:row.attr_name,
                    attr_sel:row.attr_sel,
                    attr_vals:row.attr_vals.join(' ')
                })
                if(res.meta.status !== 200) return this.$message.error('修改失败！')
                this.$message.success('操作成功！')
            },
            showInput(row){
                row.inputVisible = true
                this.$nextTick(_ => {
                this.$refs.saveTagInput.$refs.input.focus();
            });
            },
            deleteNewTag(i,row){
                row.attr_vals.splice(i,1)
                this.saveTagParams(row)
            }  
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
            },
            titleText(){
                if(this.activeName === 'many'){
                    return '动态参数'
                }
                return '静态属性'
            }
        },

    }

</script>

<style lang="less" scoped>
    .cat_opt{
        margin: 15px 0 ;
    }
    .el-tag{
        margin: 10px;
    }
    .input-new-tag{
        width: 120px;
    }
</style>