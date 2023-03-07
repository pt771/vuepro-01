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
        <!-- 卡片头部 -->
        <el-row :gutter="20">
            <el-col :span="8">
                <el-input placeholder="请输入内容" v-model="paramsInfo.query" clearable @clear="getGoodsList()">
                    <el-button slot="append" icon="el-icon-search" @click="getGoodsList()"></el-button>
                </el-input>
            </el-col>
            <el-col :span="4">
                <el-button type="primary" @click="addGoods">添加商品</el-button>
            </el-col>
        </el-row>
        <!-- 列表区域 -->
        <el-table :data="goodsList"  border stripe>
            <el-table-column type="index"></el-table-column>
            <el-table-column  prop="goods_name"  label="商品名称" ></el-table-column>
            <el-table-column  prop="goods_price"  label="商品价格（元）" width="90"></el-table-column>
            <el-table-column  prop="goods_weight"  label="商品重量" width="70"></el-table-column>
            <el-table-column  prop="add_time"  label="创建时间" width="160">
                <template slot-scope="scope">
                    {{ scope.row.add_time | dateFormat}}
                </template>
            </el-table-column>
            <el-table-column   label="操作"  width="120">
                <template slot-scope="scope">
                    <!-- 编辑按钮 -->
                    <el-button type="primary" icon="el-icon-edit" size="mini" >
                    </el-button>
                    <!-- 删除按钮 -->
                    <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row.goods_id)">
                    </el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- 页码区域 -->
        <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="paramsInfo.pagenum"
                :page-sizes="[10,20,50,100]"
                :page-size="paramsInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </el-card>
    </div>
</template>

<script>
export default {

  data() {
    return {
      /* 传递的查询参数 */
      paramsInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      /* 保存获取的数据列表 */
      goodsList: [],
      /* 获取的数据总数 */
      total: 0

    }
  },

  created() {
    this.getGoodsList()
  },

  methods: {

    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', { params: this.paramsInfo })
      if (res.meta.status !== 200) return this.$message.error('获取商品列表失败！')
      this.goodsList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.paramsInfo.pagesize = newSize
      this.getGoodsList()
    },
    handleCurrentChange(newPage) {
      this.paramsInfo.pagenum = newPage
      this.getGoodsList()
    },
    async removeById(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) return this.$message.error('删除失败！')
      this.$message.success('删除成功！')
      this.getGoodsList()
    },
    /* 跳转到添加商品页面 */
    addGoods() {
      this.$router.push('/goods/add')
    }

  }

}
</script>

<style lang="less" scoped>

</style>
