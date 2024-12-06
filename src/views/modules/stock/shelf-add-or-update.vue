<template>
  <el-dialog
    :close-on-click-modal="false"
    :title="!dataForm.id ? '新增' : '修改'"
    :visible.sync="visible"
    @closed="dialogClose"
  >
    <el-form
      ref="dataForm"
      :model="dataForm"
      :rules="dataRule"
      label-width="120px"
      @keyup.enter.native="dataFormSubmit()"
    >
      <el-form-item label="货架名" prop="shelfName">
        <el-input v-model="dataForm.shelfName" placeholder="货架名"></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input v-model="dataForm.sort" placeholder="排序"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="descript">
        <el-input v-model="dataForm.descript" placeholder="描述"></el-input>
      </el-form-item>
      <el-form-item label="货架图标" prop="icon">
        <el-input v-model="dataForm.icon" placeholder="货架图标"></el-input>
      </el-form-item>
      <el-form-item label="所属分区" prop="wlId">
        <!-- <el-input v-model="dataForm.wlId" placeholder="所属分类id"></el-input> @change="handleChange" -->
        <!-- <el-cascader filterable placeholder="试试搜索：手机" v-model="warelocationPath" :options="categorys"  :props="props"></el-cascader> -->
        <!-- :warelocationPath="warelocationPath"自定义绑定的属性，可以给子组件传值 -->
        <category-cascader :warelocationPath.sync="warelocationPath"></category-cascader>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import CategoryCascader from '../common/category-cascader'

export default {
  data () {
    return {
      props: {
        value: 'catId',
        label: 'name',
        children: 'children'
      },
      visible: false,
      categorys: [],
      warelocationPath: [],
      dataForm: {
        shelfId: 0,
        shelfName: '',
        sort: '',
        descript: '',
        icon: '',
        wlId: 0
      },
      dataRule: {
        shelfName: [
          {required: true, message: '货架名不能为空', trigger: 'blur'}
        ],
        sort: [{required: false, message: '排序不能为空', trigger: 'blur'}],
        descript: [
          {required: true, message: '描述不能为空', trigger: 'blur'}
        ],
        icon: [{required: false, message: '组图标不能为空', trigger: 'blur'}],
        wlId: [
          {required: true, message: '所属分类id不能为空', trigger: 'blur'}
        ]
      }
    }
  },
  components: {CategoryCascader},

  methods: {
    dialogClose () {
      this.warelocationPath = []
    },
    getCategorys () {
      this.$http({
        url: this.$http.adornUrl('/stock/warelocation/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.categorys = data.data
      })
    },
    init (id) {
      this.dataForm.shelfId = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.shelfId) {
          this.$http({
            url: this.$http.adornUrl(
              `/stock/shelf/info/${this.dataForm.shelfId}`
            ),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.shelfName = data.data.shelfName
              this.dataForm.sort = data.data.sort
              this.dataForm.descript = data.data.descript
              this.dataForm.icon = data.data.icon
              this.dataForm.wlId = data.data.wlId
              //查出wlId的完整路径
              this.warelocationPath = data.data.warelocationPath
            }
          })
        }
      })
    },
    // 表单提交
    dataFormSubmit () {
      this.$refs['dataForm'].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/stock/shelf/${
                !this.dataForm.shelfId ? 'save' : 'update'
              }`
            ),
            method: 'post',
            data: this.$http.adornData({
              shelfId: this.dataForm.shelfId || undefined,
              shelfName: this.dataForm.shelfName,
              sort: this.dataForm.sort,
              descript: this.dataForm.descript,
              icon: this.dataForm.icon,
              wlId: this.warelocationPath[this.warelocationPath.length - 1]
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.visible = false
                  this.$emit('refreshDataList')
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }
      })
    }
  },
  created () {
    this.getCategorys()
  }
}
</script>
