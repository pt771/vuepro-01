<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <!-- 添加角色按钮 -->
            <el-row>
                <el-col>
                    <el-button type="primary" style= "margin-bottom :15px">添加角色</el-button>
                </el-col>
            </el-row>
            <!-- 角色列表 -->
            <el-table :data="rolesList"  border  stripe>
                <el-table-column label="展开" type="expand">
                    <template slot-scope="scope">
                        <el-row v-for = "(item1,i1) in scope.row.children" :key="item1.id"
                        :class="[ 'velcenter','bdbottom', i1 === 0 ? 'bdtop' : '']" >
                            <!-- 渲染一级权限 -->
                            <el-col :span="5" :class="'tcenter'">
                                <el-tag >{{ item1.authName }}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 渲染二级、三级权限 -->
                            <el-col :span="19">
                                <el-row v-for="(item2,i2) in item1.children" :key = "item2.id"
                                :class="[i2 === 0 ? '' : 'bdtop', 'velcenter']">
                                <!-- 渲染二级菜单 -->
                                    <el-col :span="6">
                                        <el-tag type="success">{{ item2.authName }}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                <!-- 渲染三级菜单 -->
                                    <el-col :span="18">
                                        <el-tag type="warning" v-for="(item3,i3) in item2.children"
                                        :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">{{ item3.authName }}</el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column label="序号" type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="测试角色" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" width="300px">
                    <template slot-scope="scope">
                        <!-- 修改 -->
                        <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                        <!-- 删除 -->
                        <el-button type="danger" icon="el-icon-delete" size="mini" >删除</el-button>
                        <!-- 角色分配 -->
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
          <!-- 分配权限对话框 -->
          <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
            <el-tree :data="rightsList" :props="rightsListProps" show-checkbox
            node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef">

            </el-tree>
            <span slot="footer" class="dialog-footer">
            <el-button @click="setRightDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="saveRights">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
  data() {
    return {
      rolesList: [],
      setRightDialogVisible: false,
      rightsList: [],
      rightsListProps: {
        children: 'children',
        label: 'authName'
      },
      defKeys: [],
      roleId: []
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status != 200) return this.$message.error('获取权限角色列表失败！')
      this.rolesList = res.data
    },
    async removeRightById(role, rightId) {
      const confirmResult = await this.$confirm('此操作将删除本条信息，是否继续？',
        '提示', { confirmButtonText: '确定', cancelButtonText: '取消', type: 'warning' }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status != 200) return this.$message.error('删除失败！')
      this.$message.success('删除用户成功！')
      role.children = res.data
    },
    async showSetDialog(role) {
      /*  获取当前角色Id */
      this.roleId = role.id
      /* 获取所有权限数据 */
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取失败')
      this.rightsList = res.data
      this.getdefKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    /* 获取三级权限的id */
    getdefKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item =>
        this.getdefKeys(item, arr)
      )
    },
    setRightDialogClosed() {
      this.defKeys = []
    },
    async saveRights() {
      const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
      const IdStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: IdStr })
      if (res.meta.status !== 200) return this.$message.error('添加权限失败！')
      this.$message.success('添加权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    }

  }
}
</script>

<style lang="less" scoped>
  .el-tag{
    margin: 7px;

  }
  .bdtop{
    border-top: 1px solid #eee;
  }
  .bdbottom{
    border-bottom: 1px solid #eee;
  }
  .velcenter{
    display: flex ;
    align-items: center ;
  }
  .tcenter{
    text-align: center;
  }
</style>
