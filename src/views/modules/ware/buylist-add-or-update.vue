<template>
  <el-dialog
    :close-on-click-modal="false"
    :title="!dataForm.buyListId ? '新增' : '修改'"
    :visible.sync="visible">
    <el-form ref="dataForm" :model="dataForm" :rules="dataRule" label-width="120px"
             @keyup.enter.native="dataFormSubmit()">
      <el-form-item label="优先级" prop="priority">
        <el-input v-model="dataForm.priority" placeholder="优先级"></el-input>
      </el-form-item>
      <el-form-item label="采购单名称" prop="priority">
        <el-input v-model="dataForm.buyListName" placeholder="采购单名称"></el-input>
      </el-form-item>
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
      dataForm: {
        buyListId: 0,
        assigneeId: '',
        assigneeName: '',
        phone: '',
        priority: '',
        status: 0,
        buyListName: '',
        createTime: '',
        updateTime: ''
      },
      dataRule: {
        assigneeId: [
          {required: true, message: '采购人id不能为空', trigger: 'blur'}
        ],
        assigneeName: [
          {required: true, message: '采购人名不能为空', trigger: 'blur'}
        ],
        phone: [
          {required: true, message: '联系方式不能为空', trigger: 'blur'}
        ],
        priority: [
          {required: true, message: '优先级不能为空', trigger: 'blur'}
        ],
        status: [
          {required: true, message: '状态不能为空', trigger: 'blur'}
        ],
        createTime: [
          {required: true, message: '创建日期不能为空', trigger: 'blur'}
        ],
        updateTime: [
          {required: true, message: '更新日期不能为空', trigger: 'blur'}
        ]
      }
    }
  },
  methods: {
    init (buyListId) {
      this.dataForm.buyListId = buyListId || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.buyListId) {
          this.$http({
            url: this.$http.adornUrl(`/purchase/buylist/info/${this.dataForm.buyListId}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.assigneeId = data.data.assigneeId
              this.dataForm.assigneeName = data.data.assigneeName
              this.dataForm.phone = data.data.phone
              this.dataForm.priority = data.data.priority
              this.dataForm.status = data.data.status
              this.dataForm.createTime = data.data.createTime
              this.dataForm.updateTime = data.data.updateTime
              this.dataForm.buyListName = data.data.buyListName
            }
          })
        }
      })
    },
    // 表单提交
    dataFormSubmit () {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(`/purchase/buylist/${!this.dataForm.buyListId ? 'save' : 'update'}`),
            method: 'post',
            data: this.$http.adornData({
              'buyListId': this.dataForm.buyListId || undefined,
              'assigneeId': this.dataForm.assigneeId,
              'assigneeName': this.dataForm.assigneeName,
              'phone': this.dataForm.phone,
              'priority': this.dataForm.priority,
              'status': this.dataForm.status,
              'createTime': this.dataForm.createTime,
              'updateTime': this.dataForm.updateTime,
              'buyListName': this.dataForm.buyListName
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
