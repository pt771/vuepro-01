<template>
    <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>添加商品</el-breadcrumb-item>
        </el-breadcrumb>
    <!-- 卡片试图 -->
    <el-card>
        <!-- 警告区域 -->
        <el-alert title="添加商品信息" type="info" center show-icon :closable=false></el-alert>
        <!-- 步骤条 -->
        <el-steps :active="activeStepNum  - 0" align-center :space="200" finish-status="success" >
            <el-step title="基本信息" ></el-step>
            <el-step title="商品参数" ></el-step>
            <el-step title="商品属性" ></el-step>
            <el-step title="商品图片" ></el-step>
            <el-step title="商品内容" ></el-step>
            <el-step title="完成" ></el-step>
        </el-steps>

        <!-- form表单 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
        <!-- tab栏区域 -->
            <el-tabs :tab-position="'left'"  v-model="activeStepNum" :before-leave="beforTabLeave" @tab-click="tabClicked">
                <el-tab-pane label="基本信息" name="0">
                    <el-form-item label="商品名称" prop="goods_name">
                        <el-input v-model="addForm.goods_name"></el-input>
                    </el-form-item>
                    <el-form-item label="商品价格" prop="goods_price">
                        <el-input v-model="addForm.goods_price" type="number"></el-input>
                    </el-form-item>
                    <el-form-item label="商品重量" prop="goods_weight">
                        <el-input v-model="addForm.goods_weight" type="number"></el-input>
                    </el-form-item>
                    <el-form-item label="商品数量" prop="goods_number">
                        <el-input v-model="addForm.goods_number" type="number"></el-input>
                    </el-form-item>
                    <el-form-item label="商品分类" prop="goods_cat">
                        <el-cascader v-model="addForm.goods_cat" :options="cateList" :props="{ expandTrigger:'hover', ...cateProps  }"
                        @change="handleChange">

                        </el-cascader>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane label="商品参数" name="1">
                    <el-form-item :label="item.attr_name" v-for="item in manyTabData" :key="item.attr_id">
                        <el-checkbox-group v-model="item.attr_vals">
                            <el-checkbox :label="cb" v-for="(cb,i) in item.attr_vals" :key="i" border></el-checkbox>
                        </el-checkbox-group>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane label="商品属性" name="2">
                    <el-form-item :label="item.attr_name" v-for="item in onlyTabData" :key="item.attr_id">
                        <el-input v-model="item.attr_vals"></el-input>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane label="商品图片" name="3">
                    <el-upload
                         :action="uploadUrl"
                         :on-preview="handlePreview"
                         :on-remove="handleRemove"
                         :on-success="handleSuccess"
                         list-type="picture" :headers="headersObj">
                        <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                    </el-upload>
                </el-tab-pane>
                <el-tab-pane label="商品内容" name="4">
                    <!-- 富文本区域 -->
                    <quill-editor
                        v-model="addForm.goods_introduce">
                    </quill-editor>
                    <!-- 添加商品按钮 -->
                    <el-button type="primary" @click="add">
                        添加商品
                    </el-button>
                </el-tab-pane>
            </el-tabs>

        </el-form>
        <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%" >
            <img :src="previewPath" alt="" class="previewImg" >
        </el-dialog>
    </el-card>
    </div>
</template>

<script>
import _ from 'lodash'

export default {

  data() {
    return {
      activeStepNum: '0',
      cateList: [],
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      addForm: {
        goods_name: '',
        goods_weight: 0,
        goods_price: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请选择商品分类', trigger: 'blur' }
        ]
      },

      manyTabData: [],
      onlyTabData: [],
      uploadUrl: 'http://127.0.0.1:8888/api/private/v1/upload',
      headersObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewPath: '',
      previewVisible: false

    }
  },

  created() {
    this.getCateList()
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  },

  methods: {

    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取数据列表失败！')
      this.cateList = res.data
    },
    handleChange() {
      console.log(this.addForm.goods_cat)
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    beforTabLeave(activeName, oldActiveName) {
      if (oldActiveName = '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类！')
        return false
      }
    },
    async tabClicked() {
      /* 商品参数标签面 */
      if (this.activeStepNum === '1') {
        const { data: res } = await this.$http(`categories/${this.cateId}/attributes`, {
          params: { sel: 'many' }
        })
        if (res.meta.status !== 200) return this.$message.error('获取参数失败！')
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTabData = res.data
      }
      /* 商品属性标签页面 */

      else if (this.activeStepNum === '2') {
        const { data: res } = await this.$http(`categories/${this.cateId}/attributes`, {
          params: { sel: 'only' }
        })
        if (res.meta.status !== 200) return this.$message.error('获取参数失败！')

        this.onlyTabData = res.data
      }

    },
    /* 图片上传操作 */
    handlePreview(file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    /* 移除图片操作 */
    handleRemove(file) {
      const filePath = file.response.data.tem_path
      const i = this.addForm.pics.findIndex(x =>
        x.pic === filePath
      )
      this.addForm.pics.splice(i, 1)
    },
    /* 监听图片上传成功事件 */
    handleSuccess(response) {
      const picInfo = { pic: response.data.tem_path }
      this.addForm.pics.push(picInfo)
    },
    add() {
      this.$refs.addFormRef.validate(
        async valid => {
          if (!valid) {
            return this.$message.error('请填写必要的表单项！')
          }
          /* 深拷贝表单 */
          const form = _.cloneDeep(this.addForm)
          form.goods_cat = form.goods_cat.join(',')
          /* 获取attrs属性 */
          this.manyTabData.forEach(item => {
            const newInfo1 = { attr_id: item.attr_id, attr_vals: item.attr_vals.join(' ') }
            this.addForm.attrs.push(newInfo1)
          })
          this.onlyTabData.forEach(item => {
            const newInfo2 = { attr_id: item.attr_id, attr_vals: item.attr_vals }
            this.addForm.attrs.push(newInfo2)
          })
          /* 将attrs传给拷贝对象 */
          form.attrs = this.addForm.attrs

          const { data: res } = await this.$http.post('goods', form)

          if (res.meta.status !== 201) return this.$message.error('添加商品失败！')
          this.$message.success('添加成功！')
          this.$router.push('/goods')
        }
      )
    }

  }

}
</script>

<style lang="less" scoped>
    .el-checkbox{
        margin: 0 10px 0 0 !important;
    }
    .previewImg{
        width: 100%;
    }
    .el-button{
        margin-top: 15px;
    }
</style>
