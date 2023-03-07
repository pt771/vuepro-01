<template>
    <div>
         <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>订单管理</el-breadcrumb-item>
            <el-breadcrumb-item>订单列表</el-breadcrumb-item>
        </el-breadcrumb>
    <!-- 卡片试图 -->
        <el-card>
        <!-- 卡片头部 -->
        <el-row :gutter="20">
            <el-col :span="8">
                <el-input placeholder="请输入内容" v-model="paramsInfo.query" clearable @clear="getOrderList()">
                    <el-button slot="append" icon="el-icon-search" @click="getOrderList()"></el-button>
                </el-input>
            </el-col>
        </el-row>
        <!-- 列表区域 -->
        <el-table :data="orderList"  border stripe>
            <el-table-column type="index"></el-table-column>
            <el-table-column  prop="order_number"  label="订单编号" ></el-table-column>
            <el-table-column  prop="order_price"  label="订单价格（元）" width="120"></el-table-column>
            <el-table-column  prop="pay_status"  label="是否付款" width="80">
                <template slot-scope="scope">
                    <el-tag v-if="scope.row.pay_status === '0'" type="warning">未付款</el-tag>
                    <el-tag v-else type="success">已付款</el-tag>
                </template>
            </el-table-column>
            <el-table-column  prop="is_send"  label="是否发货" width="80"></el-table-column>
            <el-table-column  prop="create_time"  label="下单时间" width="160">
                <template slot-scope="scope">
                    {{ scope.row.create_time | dateFormat}}
                </template>
            </el-table-column>
            <el-table-column   label="操作"  width="120">
                <template slot-scope="scope">
                    <!-- 编辑按钮 -->
                    <el-button type="primary" icon="el-icon-edit" size="mini" @click="showBox">
                    </el-button>
                    <!-- 删除按钮 -->
                    <el-button type="success" icon="el-icon-location" size="mini" @click="showProcessBox">
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
        <!-- 修改地址对话框 -->
        <el-dialog title="修改地址" :visible.sync="setAddressDialogVisible" width="50%" @close="setCateDialogVisibleClosed">
            <el-form :model="addAddressForm" :rules="addAddressrules" ref="addAddressFormRef" label-width="100px">
                <el-form-item label="省市区/县：" prop="address1">
                    <el-cascader v-model="addAddressForm.address1" :options="cityData"  clearable :props="{ expandTrigger: 'hover' }">
                  </el-cascader> 
                </el-form-item>
                <el-form-item label="详细地址：" prop="address2">
                  <el-input v-model="addAddressForm.address2" ></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
            <el-button @click="setAddressDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 物流进度对话框 -->
        <el-dialog title="物流进度" :visible.sync="setProcessDialogVisible" width="50%">
                <el-timeline >
                    <el-timeline-item
                    v-for="(activity, index) in processInfo"
                    :key="index"
                    :timestamp="activity.time">
                    {{activity.context}}
                    </el-timeline-item>
                </el-timeline>
        </el-dialog>
    </div>
</template>

<script>

import cityData from './citydata.js'

export default {

  data() {
    return {
        /* 请求参数 */
        paramsInfo:{
            pagesize:10,
            pagenum:1,
            query:''
        },
        /* 获取参数 */
        orderList:[],
        total:0,
        /* 对话框参数 */
        setAddressDialogVisible:false,
        addAddressForm:{
            address1:[],
            address2:''
        },
        addAddressrules:{
        address1: [
          { required: true, message: '请选择省市区/县', trigger: 'blur' },
        ],
        address2:[
          { required: true, message: '请填写详细地址', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ]
        },
        cityData:cityData,
        /* 物流对话框参数 */
        setProcessDialogVisible:false,
        processInfo:[]
    }
  },

  created() {
    this.getOrderList()
  },

  methods: {
    async getOrderList(){
        const {data:res} = await this.$http.get('orders',{params: this.paramsInfo})
        if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
        console.log(res);
        this.$message.success('获取订单列表成功！')
        this.orderList = res.data.goods
        this.total = res.data.total
    },
    /* 操作页码 */
    handleSizeChange(newSize) {
      this.paramsInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange(newPage) {
      this.paramsInfo.pagenum = newPage
      this.getOrderList()
    },
    /* 展示修改地址对话框 */
    showBox(){
        this.setAddressDialogVisible = true
    },
    setCateDialogVisibleClosed(){
        this.$refs.addAddressFormRef.resetFields()
    },
    /* 物流对话框操作 */
    async showProcessBox(){
        const {data:res} = await this.$http.get('/kuaidi/1106975712662')
        if(res.meta.status !== 200) return this.$message.error('获取物流进度失败！')
        this.processInfo = res.data
        this.setProcessDialogVisible = true
    }
  }

}
</script>

<style lang="less" scoped>
.el-cascader{
    width: 100%;
}
</style>
