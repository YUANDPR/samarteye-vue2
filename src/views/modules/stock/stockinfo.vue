<template>
  <div class="mod-config">
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;"
    >
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="stockId" header-align="center" align="center" label="stockId"></el-table-column>
      <el-table-column prop="stockName" header-align="center" align="center" label="货物名称"></el-table-column>
      <el-table-column prop="stockDescription" header-align="center" align="center" label="描述"></el-table-column>
      <el-table-column prop="wlId" header-align="center" align="center" label="分类"></el-table-column>
      <el-table-column prop="supplierId" header-align="center" align="center" label="供用商"></el-table-column>
      <el-table-column prop="weight" header-align="center" align="center" label="重量"></el-table-column>
      <el-table-column prop="publishStatus" header-align="center" align="center" label="上架状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.publishStatus == 0">新入库</el-tag>
          <el-tag v-if="scope.row.publishStatus == 1">已上架</el-tag>
          <el-tag v-if="scope.row.publishStatus == 2">已出库</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="createTime" header-align="center" align="center" label="创建时间"></el-table-column>
      <el-table-column prop="updateTime" header-align="center" align="center" label="修改时间"></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.publishStatus == 0"
            type="text"
            size="small"
            @click="stockUp(scope.row.stockId)"
          >上架</el-button>
          <el-button type="text" size="small" @click="detailShow(scope.row.stockId)">详情</el-button>
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
    
      <!-- Table -->
<el-dialog title="货物详情" :visible.sync="dialogTableVisible">
  <el-table :data="gridData">
    <el-table-column property="wlId" label="分区" width="150"></el-table-column>
    <el-table-column property="shelfName" label="货架" width="150"></el-table-column>
    <el-table-column property="valueSelect" label="名称" width="200"></el-table-column>
    <el-table-column property="onestockCount" label="数量"></el-table-column>

  </el-table>
</el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      gridData: [{
      valueSelect: '',
      onestockCount: 0,
      shelfName: '',
      wlId: 0,
      }],
      dialogTableVisible: false,

      dataSub: null,
      dataForm: {},
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    };
  },
  props: {
    catId: {
      type: Number,
      default: 0
    }
  },
  components: {},
  activated() {
    this.getDataList();
  },
  methods: {
    stockUp(stockId) {
      this.$http({
        url: this.$http.adornUrl("/stock/stock/" + stockId + "/up"),
        method: "post"
      }).then(({ data }) => {
        if (data && data.data === 1) {
          this.$message({
            message: "上架成功",
            type: "success",
            duration: 1500,
            onClose: () => {
              this.getDataList();
            }
          });
        } else {
          this.$message.error("该(批)货物中存在未上架的货物单元，请先上架它们");
        }
      });
    },
      detailShow(stockId) {
        this.$http({
        url: this.$http.adornUrl(`/stock/onestock/${stockId}/info`),
        method: "get",
      }).then(({ data }) => {
          this.gridData = data.data;
      });
        this.dialogTableVisible = true;

    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      let param = {};
      Object.assign(param, this.dataForm, {
        page: this.pageIndex,
        limit: this.pageSize
      });
      this.$http({
        url: this.$http.adornUrl("/stock/stock/list"),
        method: "get",
        params: this.$http.adornParams(param)
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
    addOrUpdateHandle(stockId) {}
  },
  mounted() {
    this.dataSub = PubSub.subscribe("dataForm", (msg, val) => {
      console.log("~~~~~", val);
      this.dataForm = val;
      this.getDataList();
    });
  },
  beforeDestroy() {
    PubSub.unsubscribe(this.dataSub);
  }
};
</script>
