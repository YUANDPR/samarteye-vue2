<template>
  <el-dialog
    :close-on-click-modal="false"
    :title="!dataForm.buyDemandId ? '新增' : '修改'"
    :visible.sync="visible"
  >
    <el-form
      ref="dataForm"
      :model="dataForm"
      :rules="dataRule"
      label-width="120px"
      @keyup.enter.native="dataFormSubmit()"
    >
      <el-form-item label="采购品id" prop="warestockId">
        <el-input v-model="dataForm.warestockId" placeholder="采购品id"></el-input>
      </el-form-item>
      <el-form-item label="采购品名称" prop="buyNum">
        <el-input v-model="dataForm.valueSelect" placeholder="采购品名称"></el-input>
      </el-form-item>
      <el-form-item label="采购数量" prop="buyNum">
        <el-input v-model="dataForm.buyNum" placeholder="采购数量"></el-input>
      </el-form-item>

      <!-- [0新建，1已分配，2正在采购，3已完成，4采购失败] -->
      <!-- <el-form-item label="状态" prop="status">
        <el-select v-model="dataForm.status" placeholder="请选择状态" clearable>
          <el-option label="新建" :value="0"></el-option>
          <el-option label="已分配" :value="1"></el-option>
          <el-option label="正在采购" :value="2"></el-option>
          <el-option label="已完成" :value="3"></el-option>
          <el-option label="采购失败" :value="4"></el-option>
        </el-select>
      </el-form-item>-->
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data () {
    return {
      visible: false,
      wareList: [],
      dataForm: {
        buyDemandId: 0,
        buyListId: '',
        warestockId: '',
        buyNum: '',
        valueSelect: '',
        origin: 0,
        status: 0
      },
      dataRule: {
        warestockId: [
          {required: true, message: '采购商品id不能为空', trigger: 'blur'}
        ],
        buyNum: [
          {required: true, message: '采购数量不能为空', trigger: 'blur'}
        ],
        wareId: [{required: true, message: '仓库id不能为空', trigger: 'blur'}]
      }
    }
  },
  created () {
    this.getWares()
  },
  methods: {
    getWares () {
      this.$http({
        url: this.$http.adornUrl('/ware/wareinfo/list'),
        method: 'get',
        params: this.$http.adornParams({
          page: 1,
          limit: 500
        })
      }).then(({data}) => {
        this.wareList = data.page.list
      })
    },
    init (buyDemandId) {
      this.dataForm.buyDemandId = buyDemandId || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.buyDemandId) {
          this.$http({
            url: this.$http.adornUrl(
              `/purchase/buydemand/info/${this.dataForm.buyDemandId}`
            ),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.buyListId = data.data.buyListId
              this.dataForm.warestockId = data.data.warestockId
              this.dataForm.buyNum = data.data.buyNum
              this.dataForm.origin = data.data.origin
              this.dataForm.valueSelect = data.data.valueSelect
              this.dataForm.status = data.data.status
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
              `/purchase/buydemand/${!this.dataForm.buyDemandId ? 'save' : 'update'}`
            ),
            method: 'post',
            data: this.$http.adornData({
              buyDemandId: this.dataForm.buyDemandId || undefined,
              buyListId: this.dataForm.buyListId,
              warestockId: this.dataForm.warestockId,
              buyNum: this.dataForm.buyNum,
              origin: this.dataForm.origin,
              valueSelect: this.dataForm.valueSelect,
              status: this.dataForm.status
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
  }
}
</script>
