<template>
  <el-dialog
    :title="!dataForm.askmbillId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="叫料人员id" prop="eeId">
      <el-input v-model="dataForm.eeId" placeholder="叫料人员id"></el-input>
    </el-form-item>
    <el-form-item label="描述" prop="askmbillDescription">
      <el-input v-model="dataForm.askmbillDescription" placeholder="描述"></el-input>
    </el-form-item>
    <el-form-item label="叫料位置id" prop="asklocationId">
      <el-input v-model="dataForm.asklocationId" placeholder="叫料位置id"></el-input>
    </el-form-item>
    <el-form-item label="叫料处理分区id" prop="wlId">
      <el-input v-model="dataForm.wlId" placeholder="叫料处理分区id"></el-input>
    </el-form-item>
    <el-form-item label="叫料位置名" prop="asklocationName">
      <el-input v-model="dataForm.asklocationName" placeholder="叫料位置名"></el-input>
    </el-form-item>
    <el-form-item label="货物种类值" prop="valueSelect">
      <el-input v-model="dataForm.valueSelect" placeholder="货物种类值"></el-input>
    </el-form-item>
    <el-form-item label="数量" prop="count">
      <el-input v-model="dataForm.count" placeholder="数量"></el-input>
    </el-form-item>
    <el-form-item label="状态" prop="status">
        <el-select style="width:160px" v-model="dataForm.status" clearable>
              <el-option label="申请中" :value="0"></el-option>
              <el-option label="已撤回" :value="2"></el-option>
              <el-option label="已完成" :value="1"></el-option>
            </el-select>
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
          askmbillId: 0,
          eeId: '',
          askmbillDescription: '',
          asklocationId: '',
          wlId: '',
          asklocationName: '',
          valueSelect: '',
          count: '',
          status: '',
          createTime: '',
          updateTime: ''
        },
        dataRule: {
         
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.askmbillId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.askmbillId) {
            this.$http({
              url: this.$http.adornUrl(`/material/askmbill/info/${this.dataForm.askmbillId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.eeId = data.askmbill.eeId
                this.dataForm.askmbillDescription = data.askmbill.askmbillDescription
                this.dataForm.asklocationId = data.askmbill.asklocationId
                this.dataForm.wlId = data.askmbill.wlId
                this.dataForm.asklocationName = data.askmbill.asklocationName
                this.dataForm.valueSelect = data.askmbill.valueSelect
                this.dataForm.count = data.askmbill.count
                this.dataForm.status = data.askmbill.status
                this.dataForm.createTime = data.askmbill.createTime
                this.dataForm.updateTime = data.askmbill.updateTime
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
              url: this.$http.adornUrl(`/material/askmbill/${!this.dataForm.askmbillId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'askmbillId': this.dataForm.askmbillId || undefined,
                'eeId': this.dataForm.eeId,
                'askmbillDescription': this.dataForm.askmbillDescription,
                'asklocationId': this.dataForm.asklocationId,
                'wlId': this.dataForm.wlId,
                'asklocationName': this.dataForm.asklocationName,
                'valueSelect': this.dataForm.valueSelect,
                'count': this.dataForm.count,
                'status': this.dataForm.status,
                'createTime': this.dataForm.createTime,
                'updateTime': this.dataForm.updateTime
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
