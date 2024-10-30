<template>

  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      
      <el-form-item label="分区">
  
             
      <el-select style="width:160px;" v-model="dataForm.wlppId" placeholder="请选择分区" clearable>
          <el-option :label="w.name" :value="w.wlId" v-for="w in wareList" :key="w.id"></el-option>
    
       </el-select>  
      </el-form-item>
      <el-form-item label="货架">
        <el-input v-model="dataForm.shelfName" placeholder="货架" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <!-- 暂时不允许手动新增库存 -->
      <!--  <el-button v-if="isAuth('ware:waresku:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>  -->
        <el-button
          v-if="isAuth('ware:waresku:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
        >批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;"
    >
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="warematerialId" header-align="center" align="center" label="warematerialId"></el-table-column>
      <el-table-column prop="wlppId" header-align="center" align="center" label="厂区"></el-table-column>
      <el-table-column prop="wlpId" header-align="center" align="center" label="车间"></el-table-column>
      <el-table-column prop="wlId" header-align="center" align="center" label="分区"></el-table-column>
      <el-table-column prop="shelfName" header-align="center" align="center" label="货架"></el-table-column>
      <el-table-column prop="materialTypeName" header-align="center" align="center" label="库存种类"></el-table-column>
      <el-table-column prop="valueSelect" header-align="center" align="center" label="库存种类值"></el-table-column>
      <el-table-column prop="realCount" header-align="center" align="center" label="真实库存"></el-table-column>
      <el-table-column prop="warnCount" header-align="center" align="center" label="预警库存"></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.warematerialId)">修改预警库存</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.warematerialId)">删除</el-button>
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
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>

import AddOrUpdate from "./warematerial-add-or-update";
export default {
  data() {
    return {
      wareList: [],
      dataForm: {
        wlppId: "",
        shelfName: ""
      },
      warelocationPath: [],
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    };
  },
  components: {
    AddOrUpdate
   
  },
  activated() {  //这里的skuId实际对应shelfName，但是这里是另一个页面链接过来的，我先没改
    console.log("接收到", this.$route.query.skuId);
    if (this.$route.query.skuId) {
      this.dataForm.skuId = this.$route.query.skuId;
    }
    this.getWares();
    this.getDataList();
  },
  methods: {
      getWares() {  
      this.$http({
        url: this.$http.adornUrl("/material/warelocation/list/tree"),
        method: "get",
        params: this.$http.adornParams({
          page: 1,
          limit: 500
        })
      }).then(({ data }) => {
        this.wareList = data.data;
      });
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/ware/warematerial/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          shelfName: this.dataForm.shelfName,
          wlppId: this.dataForm.wlppId
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
    },
    // 新增 / 修改
    addOrUpdateHandle(warematerialId) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(warematerialId);
      });
    },
    // 删除
    deleteHandle(warematerialId) {
      var warematerialIds = warematerialId
        ? [warematerialId]
        : this.dataListSelections.map(item => {
            return item.warematerialId;
          });
      this.$confirm(
        `确定对[warematerialId=${warematerialIds.join(",")}]进行[${warematerialId ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/ware/warematerial/delete"),
          method: "post",
          data: this.$http.adornData(warematerialIds, false)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
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
      });
    }
  }
};
</script>
