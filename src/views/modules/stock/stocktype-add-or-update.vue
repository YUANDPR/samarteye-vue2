<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
    @closed="dialogClose"
  >
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="120px">
      <!--       @keyup.enter.native="dataFormSubmit()" -->
      <el-form-item label="库存种类" prop="stockTypeName">
        <el-input v-model="dataForm.stockTypeName" placeholder="库存种类"></el-input>
      </el-form-item>
      <el-form-item label="货物类型" prop="stockTypeType">
        <el-select v-model="dataForm.stockTypeType" placeholder="请选择">
          <el-option label="基本货物" :value="1"></el-option>
          <el-option label="货架外货物" :value="0"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="值类型" prop="valueType">
        <el-switch
          v-model="dataForm.valueType"
          active-text="允许多个值"
          inactive-text="只能单个值"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :inactive-value="0"
          :active-value="1"
        ></el-switch>
      </el-form-item>
      <el-form-item label="可选值" prop="valueSelect">
        <!-- <el-input v-model="dataForm.valueSelect"></el-input> -->
        <el-select
          v-model="dataForm.valueSelect"
          multiple
          filterable
          allow-create
          placeholder="请输入内容"
        ></el-select>
      </el-form-item>
      <el-form-item label="库存种类图标" prop="icon">
        <el-input v-model="dataForm.icon" placeholder="库存种类图标"></el-input>
      </el-form-item>
      <el-form-item label="所属分区" prop="wlId">
        <category-cascader :warelocationPath.sync="warelocationPath"></category-cascader>
      </el-form-item>
      <el-form-item label="所属货架" prop="shelfId" v-if="type == 1">
        <el-select ref="groupSelect" v-model="dataForm.shelfId" placeholder="请选择">
          <el-option
            v-for="item in attrGroups"
            :key="item.shelfId"
            :label="item.shelfName"
            :value="item.shelfId"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="可检索" prop="searchType" v-if="type == 1" hidden>
        <el-switch
          v-model="dataForm.searchType"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
        ></el-switch>
      </el-form-item>
      <el-form-item label="快速展示" prop="showDesc" v-if="type == 1">
        <el-switch
          v-model="dataForm.showDesc"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
        ></el-switch>
      </el-form-item>
      <el-form-item label="启用状态" prop="enable">
        <el-switch
          v-model="dataForm.enable"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
        ></el-switch>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import CategoryCascader from "../common/category-cascader";
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        stockTypeId: 0,
        stockTypeName: "",
        searchType: 0,
        valueType: 1,
        icon: "",
        valueSelect: "",
        stockTypeType: 1,
        enable: 1,
        wlId: "",
        shelfId: "",
        showDesc: 0
      },
      warelocationPath: [],
      attrGroups: [],
      dataRule: {
        stockTypeName: [
          { required: true, message: "种类名不能为空", trigger: "blur" }
        ],
        searchType: [
          {
            required: false,
            message: "不能为空",
            trigger: "blur"
          }
        ],
        valueType: [
          {
            required: true,
            message: "值类型不能为空",
            trigger: "blur"
          }
        ],
        icon: [
          { required: false, message: "不能为空", trigger: "blur" }
        ],
        stockTypeType: [
          {
            required: true,
            message: "种类类型不能为空",
            trigger: "blur"
          }
        ],
        enable: [
          {
            required: false,
            message: "启用状态不能为空",
            trigger: "blur"
          }
        ],
        wlId: [
          {
            required: true,
            message: "需要选择正确的库存分区",
            trigger: "blur"
          }
        ],
        showDesc: [
          {
            required: false,
            message: "快速展示不能为空",
            trigger: "blur"
          }
        ]
      }
    };
  },
  props:{
    type:{
      type: Number,
      default: 1
    }
  },
  watch: {
    warelocationPath(path) {
      //监听到路径变化需要查出这个三级分类的分组信息
      console.log("路径变了", path);
      this.attrGroups = [];
      this.dataForm.shelfId = "";
      this.dataForm.wlId = path[path.length - 1];
      if (path && path.length == 3) {
        this.$http({
          url: this.$http.adornUrl(
            `/stock/stocktype/list/${path[path.length - 1]}`
          ),
          method: "get",
          params: this.$http.adornParams({ page: 1, limit: 10000000 })
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.attrGroups = data.page.list;
          } else {
            this.$message.error(data.msg);
          }
        });
      } else if (path.length == 0) {
        this.dataForm.wlId = "";
      } else {
        this.$message.error("请选择正确的分类");
        this.dataForm.wlId = "";
      }
    }
  },
  components: { CategoryCascader },
  methods: {
    init(id) {
      this.dataForm.stockTypeId = id || 0;
      this.dataForm.stockTypeType = this.type;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.stockTypeId) {
          this.$http({
            url: this.$http.adornUrl(
              `/stock/stocktype/info/${this.dataForm.stockTypeId}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.stockTypeName = data.data.stockTypeName;
              this.dataForm.searchType = data.data.searchType;
              this.dataForm.valueType = data.data.valueType;
              this.dataForm.icon = data.data.icon;
              this.dataForm.valueSelect = data.data.valueSelect.split(";");
              this.dataForm.stockTypeType = data.data.stockTypeType;
              this.dataForm.enable = data.data.enable;
              this.dataForm.wlId = data.data.wlId;
              this.dataForm.showDesc = data.data.showDesc;
              //shelfId
              //warelocationPath
              this.warelocationPath = data.data.warelocationPath;
              this.$nextTick(() => {
                this.dataForm.shelfId = data.data.shelfId;
              });
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
              `/stock/stocktype/${!this.dataForm.stockTypeId ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              stockTypeId: this.dataForm.stockTypeId || undefined,
              stockTypeName: this.dataForm.stockTypeName,
              searchType: this.dataForm.searchType,
              valueType: this.dataForm.valueType,
              icon: this.dataForm.icon,
              valueSelect: this.dataForm.valueSelect.join(";"),
              stockTypeType: this.dataForm.stockTypeType,
              enable: this.dataForm.enable,
              wlId: this.dataForm.wlId,
              shelfId: this.dataForm.shelfId,
              showDesc: this.dataForm.showDesc
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
    },
    //dialogClose
    dialogClose() {
      this.warelocationPath = [];
    }
  }
};
</script>
