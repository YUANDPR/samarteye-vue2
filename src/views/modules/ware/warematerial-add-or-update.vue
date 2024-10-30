<template>
  <el-dialog
    :title="!dataForm.warematerialId ? '新增' : '库存设置'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="120px"
    >
  <!--    <el-form-item label="sku_id" prop="skuId">
        <el-input v-model="dataForm.skuId" placeholder="sku_id"></el-input>
      </el-form-item>
      <el-form-item label="仓库" prop="wareId">
        <el-select v-model="dataForm.wareId" placeholder="请选择仓库" clearable>
          <el-option :label="w.name" :value="w.id" v-for="w in wareList" :key="w.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="库存数" prop="material">
        <el-input v-model="dataForm.material" placeholder="库存数"></el-input>
      </el-form-item>
      <el-form-item label="sku_name" prop="skuName">
        <el-input v-model="dataForm.skuName" placeholder="sku_name"></el-input>
      </el-form-item>  -->
      <el-form-item label="预警库存" prop="warnCount">
        <el-input v-model="dataForm.warnCount"></el-input>
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
  data() {
    return {
      visible: false,
      wareList: [],
      dataForm: {
        warematerialId: 0,
        skuId: "",
        wareId: "",
        material: 0,
        skuName: "",
        warnCount: 0
      },
      dataRule: {
        skuId: [{ required: true, message: "sku_id不能为空", trigger: "blur" }],
        wareId: [
          { required: true, message: "仓库id不能为空", trigger: "blur" }
        ],
        material: [{ required: true, message: "库存数不能为空", trigger: "blur" }],
        skuName: [
          { required: true, message: "sku_name不能为空", trigger: "blur" }
        ]
      }
    };
  },
  created(){
    this.getWares();
  },
  methods: {
    getWares() {
      this.$http({
        url: this.$http.adornUrl("/ware/wareinfo/list"),
        method: "get",
        params: this.$http.adornParams({
          page: 1,
          limit: 500
        })
      }).then(({ data }) => {
        this.wareList = data.page.list;
      });
    },
    init(warematerialId) {
      this.dataForm.warematerialId = warematerialId || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.warematerialId) {
          this.$http({
            url: this.$http.adornUrl(`/ware/warematerial/info/${this.dataForm.warematerialId}`),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
            //  this.dataForm.skuId = data.data.skuId;
           //   this.dataForm.wareId = data.data.wareId;
             // this.dataForm.material = data.data.material;
              //this.dataForm.skuName = data.data.skuName;
              this.dataForm.warnCount = data.data.warnCount;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/ware/warematerial/${!this.dataForm.warematerialId ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              warematerialId: this.dataForm.warematerialId || undefined,
           //   skuId: this.dataForm.skuId,
          //    wareId: this.dataForm.wareId,
            //  material: this.dataForm.material,
            //  skuName: this.dataForm.skuName,
              warnCount: this.dataForm.warnCount
            })
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                }
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    }
  }
};
</script>
