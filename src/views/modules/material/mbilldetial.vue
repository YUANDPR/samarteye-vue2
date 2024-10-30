<template>

  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form :inline="true" :model="dataForm">
        <el-form-item label="分区">
          <category-cascader :warelocationPath.sync="warelocationPath"></category-cascader>
        </el-form-item>
           <el-form-item label="状态">
            <el-select style="width:160px" v-model="dataForm.status" clearable>
              <el-option label="申请中" :value="0"></el-option>
              <el-option label="已送达" :value="2"></el-option>
            </el-select>
          </el-form-item>

        <el-form-item label="检索">
          <el-input style="width:160px" v-model="dataForm.key" clearable></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="searchSkuInfo">查询</el-button>
        </el-form-item>
      </el-form>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;"
      @expand-change="getSkuDetails"
    >
      <el-table-column type="expand">
        <template slot-scope="scope">
          描述：{{scope.row.mbillDescription}}
          <br />
          分区ID：{{scope.row.wlId}}
          <br />
          所属申请单ID：{{scope.row.mbillId}}
          <br />
          货物种类ID：{{scope.row.materialTypeId}}
          <br />
          权限等级：{{scope.row.levelNumb}}
          <br />
        </template>
      </el-table-column>
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="mbilldetailId" header-align="center" align="center" label="mbilldetailId"></el-table-column>
      <el-table-column prop="shelfName" header-align="center" align="center" label="货架"></el-table-column>
      <el-table-column prop="valueSelect" header-align="center" align="center" label="货物名称"></el-table-column>
   <!--   <el-table-column prop="onematerialImage" header-align="center" align="center" label="图片">
        <template slot-scope="scope">
          <img :src="scope.row.onematerialImage" style="width:80px;height:80px;" />
        </template>
      </el-table-column>  -->
      <el-table-column prop="mbilldetailCount" header-align="center" align="center" label="申请数量"></el-table-column>
      <el-table-column prop="status" header-align="center" align="center" label="申请状态">
        <template slot-scope="scope">
              <el-tag v-if="scope.row.status == 0" >申请中</el-tag>
          <el-tag v-if="scope.row.status == 2" type="success">已送达</el-tag>
           <el-tag v-if="scope.row.status == 3" type="danger">拒绝申请</el-tag>
          <el-tag v-if="scope.row.status == 4" type="warning">撤回申请</el-tag>
             
        </template>
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
           <el-button
            v-if="scope.row.status == 0"
            type="text"
            size="small"
            @click="onematerialUp(scope.row.mbilldetailId)"
          >确认送达</el-button>

          <el-dropdown
            @command="handleCommand(scope.row,$event)"
            size="small"
            split-button
            type="text"
          >
            更多
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="materialSettings">库存管理</el-dropdown-item>
              <el-dropdown-item command="levelNumbSettings">叫料权限</el-dropdown-item>
        
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    ></el-pagination>
  </div>
</template>

<script>
import CategoryCascader from "../common/material/category-cascader";
import BrandSelect from "../common/material/supplier-select";
export default {
  data() {
    return {
      catPathSub: null,
      supplierIdSub: null,
      dataForm: {
        key: "",
        supplierId: 0,
        wlId: 0,

      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      warelocationPath: []
    };
  },
  components: {
    CategoryCascader,
    BrandSelect
  },
  activated() {
    this.getDataList();
  },
  methods: {
     onematerialUp(mbilldetailId) {
      this.$http({
        url: this.$http.adornUrl("/material/mbilldetail/" + mbilldetailId + "/received"),
        method: "post"
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "已确认送达",
            type: "success",
            duration: 1500,
            onClose: () => {
              this.getDataList();
            }
          });
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    getSkuDetails(row, expand) {
      //sku详情查询
      console.log("展开某行...", row, expand);
    },
    //处理更多指令
    handleCommand(row, command) {
      console.log("~~~~~", row, command);
      if ("materialSettings" == command) {
        this.$router.push({ path: "/ware-sku", query: { mbilldetailId: row.mbilldetailId } });
      }
    },
    searchSkuInfo() {
      this.getDataList();
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/material/mbilldetail/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
          wlId: this.dataForm.wlId,
          supplierId: this.dataForm.supplierId,
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    }
  },
  mounted() {
    this.catPathSub = PubSub.subscribe("catPath", (msg, val) => {
      this.dataForm.wlId = val[val.length - 1];
    });
    this.supplierIdSub = PubSub.subscribe("supplierId", (msg, val) => {
      this.dataForm.supplierId = val;
    });
  },
  beforeDestroy() {
    PubSub.unsubscribe(this.catPathSub);
    PubSub.unsubscribe(this.supplierIdSub);
  } //生命周期 - 销毁之前
};
</script>
