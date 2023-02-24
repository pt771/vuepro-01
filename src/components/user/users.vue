<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
            
        </el-breadcrumb>
<!-- 卡片视图 -->
        <el-card class="box-card">
         <div>
            <el-row :gutter="20">
            <el-col :span="7">
                <el-input placeholder="请输入内容" v-model="paramsInfo.query" clearable  @clear="getUserList"> 
                    <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                </el-input>
            </el-col>
            <el-col :span="4">
                <el-button type="primary" @click="addUserVisible = true">添加用户</el-button>
            </el-col>
            </el-row>
            <el-table :data="userList"  border  stripe>
                <el-table-column label="序号" type="index"></el-table-column>
                <el-table-column label="姓名" prop="username"></el-table-column>
                <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                <el-table-column label="角色" prop="role_name"></el-table-column>
                <el-table-column label="状态" >
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change='userInfoState(scope.row)'> </el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="180px">
                    <template slot-scope="scope">
                        <!-- 修改 -->
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
                        <!-- 删除 -->
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
                        <!-- 角色分配 -->
                        <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="info" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="paramsInfo.pagenum"
                :page-sizes="[1,2,5,10]"
                :page-size="paramsInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
         </div>
        </el-card>
        <!-- 添加用户 -->
        <el-dialog title="添加用户" :visible.sync="addUserVisible" width="50%" @close="dialogClose">
            <el-form :model="addForm" :rules="addrules" ref="addFormRef" label-width="70px">
                <!-- 用户名 -->
                <el-form-item label="用户名" prop="username">
                <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <!-- 密码 -->
                <el-form-item label="密码" prop="password">
                <el-input type="password" v-model="addForm.password" autocomplete="off"></el-input>
                </el-form-item>
                <!-- 邮箱 -->
                <el-form-item label="邮箱" prop="email">
                <el-input  v-model="addForm.email"></el-input>
                </el-form-item>
                <!-- 手机号码 -->
                <el-form-item label="手机" prop="mobile">
                <el-input  v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
            <el-button @click="addUserVisible = false" >取 消</el-button>
            <el-button type="primary"  @click="addUser">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 修改页面对话框 -->
        <el-dialog title="修改信息" :visible.sync="EditDialogVisible" width="50%" @close="EditDialogClose">
            <el-form :model="editForm" :rules="editrules" ref="editFormRef" label-width="70px">
                <!-- 用户名 -->
                <el-form-item label="用户名" prop="username">
                <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                 <!-- 用户名 -->
                 <el-form-item label="邮箱" prop="email">
                <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <!-- 用户名 -->
                <el-form-item label="手机号" prop="mobile">
                <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
            <el-button @click="EditDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editSubmit()">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 分配角色的对话框 -->
        <el-dialog title="分配角色" :visible.sync="setRoleDialogVisible" width="50%" @close="setRoleDialogClose">
            <div>
                <p>当前用户：{{ userInfo.username }}</p>
                <p>当前角色：{{ userInfo.role_name }}</p>
                <p>当前角色：
                <el-select v-model="selectedRoleId" placeholder="请选择">
                    <el-option
                         v-for="item in rolesList"
                         :key="item.id"
                         :label="item.roleName"
                         :value="item.id">
                    </el-option>
                </el-select>
                </p>
            </div>
            <span slot="footer" class="dialog-footer">
            <el-button @click="setRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="submitRoles">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default{
        data(){
            /* 验证邮箱 */
            var checkEmail = (rule,value,cb)=>{
                const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/

                if(regEmail.test(value)){
                    return cb()
                }

                cb( new Error('请输入合法的邮箱！'))
            }
            /* 验证手机号 */
            var checkPhone = (rule,value,cb)=>{
                const regPhone = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
                if(regPhone.test(value)){
                    return cb()
                }

                cb( new Error('请输入合法的手机号！'))
            }  
            return{
                paramsInfo:{
                    query:'',
                    pagenum:1,
                    pagesize:2
                },
                userList:[],
                total:0,
                addUserVisible:false,
                EditDialogVisible : false,
                addForm:{
                    username:'',
                    password:'',
                    email:'',
                    mobile:''
                },
                addrules:{
                    username:[
                    { required: true, message: '请输入姓名', trigger: 'blur' },
                    { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                    ],
                    password:[
                    { required: true, message: '请输入密码', trigger: 'blur' },
                    { min: 6, max: 10, message: '长度在 6 到 10 个字符', trigger: 'blur' }
                    ],
                    email:[
                    { required: true, message: '请输入邮箱', trigger: 'blur' },
                    {validator: checkEmail,trigger: 'blur'}
                    ],
                    mobile:[
                    { required: true, message: '请输入号码', trigger: 'blur' },
                    {validator: checkPhone,trigger:'blur'}
                    ]
                },
                editForm:{
                },
                editrules:{
                    email:[
                    { required: true, message: '请输入修改后的邮箱', trigger: 'blur' },
                    {validator: checkEmail,trigger: 'blur'}
                    ],
                    mobile:[
                    { required: true, message: '请输入修改后的号码', trigger: 'blur' },
                    {validator: checkPhone,trigger:'blur'}
                    ]
                },
                setRoleDialogVisible:false,
                userInfo:{},
                rolesList:[],
                selectedRoleId:''
            }
        },
        created(){
            this.getUserList()
        },
        methods:{
            async getUserList(){
               
                const { data : res } = await this.$http.get('users',{ params :this.paramsInfo})
                if(res.meta.status !== 200) return this.$message.error('获取信息失败！')
                this.userList = res.data.users
                this.total = res.data.total
            },
            handleSizeChange(newSize){
                this.paramsInfo.pagesize = newSize
                this.getUserList()
            },
            handleCurrentChange(newPage){
                this.paramsInfo.pagenum = newPage
                this.getUserList()
            },
          async userInfoState(userInfo){
              const {data:res} = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
              if(res.meta.status !== 200){
                userInfo.mg_state = !userInfo.mg_state
                return this.$message.error('更新状态失败！')
              }
              this.$message.success('更新状态成功！')
            },
            dialogClose(){
                this.$refs.addFormRef.resetFields()
            },
            addUser(){
              this.$refs.addFormRef.validate(async valide => {
                if(!valide) return
                const {data:res} = await this.$http.post('users', this.addForm)
                if(res.meta.status != 201) return this.$message.error('添加用户失败！')
                this.$message.success('用户添加成功！')
                this.addUserVisible = false
                this.getUserList()
             }) 
             
            },
            async showEditDialog(id){
                this.EditDialogVisible = true  
                const {data:res} = await  this.$http.get('users/' + id)
                if(res.meta.status !==200 ) return this.$message.error('获取失败！')
                this.editForm = res.data
            },
            EditDialogClose(){
                 this.$refs.addFormRef.resetFields()
            },
            editSubmit(){
                this.$refs.editFormRef.validate(async valid =>{
                    if (!valid)  return this.$message.error('修改失败！')
                    const {data:res} = await this.$http.put("users/" + this.editForm.id,{email:this.editForm.email,mobile:this.editForm.mobile})
                    if(res.meta.status != 200) return this.$message.error('更新用户失败！')
                    /* 关闭对话框 */
                    this.EditDialogVisible = false
                    /* 重新获取数据 */
                    this.getUserList()
                    /* 提示成功 */
                    this.$message.success('修改数据成功！')
                })

            },
            /* 根据Id删除 */
           async removeUserById(id){
                /* 弹框询问 */
             const confirmResult = await this.$confirm('此操作将删除本条信息，是否继续？',
                '提示',{confirmButtonText:'确定',cancelButtonText:'取消',type:'warning'}).catch(err => err)

             if(confirmResult !== 'confirm'){
                return this.$message.info('已取消删除！')
             }

            const {data:res} = await this.$http.delete('users/'+ id)
            if(res.meta.status != 200) return this.$message.error('删除失败！')
            this.$message.success('删除用户成功！')
            this.getUserList()
            },
           async setRole(userInfo){
              this.userInfo =userInfo
              const {data:res} = await  this.$http.get('roles')
              if(res.meta.status !==200) return this.$message.error('获取失败')
                this.rolesList = res.data
                this.setRoleDialogVisible = true
            },
            async submitRoles(){
                if(!this.selectedRoleId) {
                    return this.$message.error('请选择要分配的角色！')
                }
                const {data:res} = await this.$http.put(`users/${this.userInfo.id}/role`,{rid:this.selectedRoleId})
                if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
                this.$message.success(res.meta.msg)
                this.getUserList()
                this.setRoleDialogVisible = false 
            },
            setRoleDialogClose(){
                this.selectedRoleId = ''
                this.userInfo = {}
            }
        }
    }
</script>

<style lang="less" scoped>
.el-table{
    margin-top: 15px;
    font-size: 12px;
}
.el-form-item__label {
    font-size: 10px;
    text-align: left;
}
</style>
