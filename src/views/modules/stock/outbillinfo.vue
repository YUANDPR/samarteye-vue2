<template>
  <div class="mod-config">
    <el-table
      v-loading="dataListLoading"
      :data="dataList"
      border
      style="width: 100%;"
      @selection-change="selectionChangeHandle"
    >
      <el-table-column align="center" header-align="center" type="selection" width="50"></el-table-column>
      <el-table-column align="center" header-align="center" label="outbillId" prop="outbillId"></el-table-column>
      <el-table-column align="center" header-align="center" label="出库单名称" prop="outbillName"></el-table-column>
      <el-table-column align="center" header-align="center" label="描述" prop="outbillDescription"></el-table-column>
      <el-table-column align="center" header-align="center" label="分区" prop="wlId"></el-table-column>
      <el-table-column align="center" header-align="center" label="出库类型" prop="outtype">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.outtype == 1">物料申请单出库</el-tag>
          <el-tag v-if="scope.row.outtype == 2">手动出库</el-tag>
          <el-tag v-if="scope.row.outtype == 3">其他出库</el-tag>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" label="状态" prop="status">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status == 0">新建</el-tag>
          <el-tag v-if="scope.row.status == 1" type="info">已出库</el-tag>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" label="生成时间" prop="createTime"></el-table-column>
      <el-table-column align="center" header-align="center" label="更新时间" prop="updateTime"></el-table-column>
      <el-table-column align="center" fixed="right" header-align="center" label="操作" width="150">
        <template slot-scope="scope">
          <el-button
            size="small"
            type="text"
            @click="printbill(scope.row.outbillId)"
          >打印出库单
          </el-button>
          <el-button size="small" type="text" @click="detailShow(scope.row.outbillId)">详情</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      :current-page="pageIndex"
      :page-size="pageSize"
      :page-sizes="[10, 20, 50, 100]"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
    ></el-pagination>

    <!-- Table -->
    <el-dialog :visible.sync="dialogTableVisible" title="出库单详情">
      <el-table :data="gridData">
        <el-table-column label="分区" property="wlId" width="150"></el-table-column>
        <el-table-column label="货架" property="shelfName" width="150"></el-table-column>
        <el-table-column label="名称" property="valueSelect" width="200"></el-table-column>
        <el-table-column label="数量" property="outbilldetailCount"></el-table-column>

      </el-table>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      gridData: [{
        valueSelect: '',
        outbilldetailCount: 0,
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
    }
  },
  props: {
    catId: {
      type: Number,
      default: 0
    }
  },
  components: {},
  activated () {
    this.getDataList()
  },
  methods: {
    detailShow (outbillId) {
      this.$http({
        url: this.$http.adornUrl(`/stock/outbilldetail/${outbillId}/info`),
        method: 'get',
      }).then(({data}) => {
        this.gridData = data.data
      })
      this.dialogTableVisible = true

    },
    printbill (outbillId) {
      alert('打印未开放，请联系管理员')
      /** this.$http({
       url: this.$http.adornUrl("/stock/outbill/" + outbillId + "/revoke"),
       method: "post"
       }).then(({ data }) => {

       });  */
    },
    attrUpdateShow (row) {
      console.log(row)
      this.$router.push({
        path: '/material-materialtypeupdate',
        query: {spuId: row.outbillId, wlId: row.wlId}
      })
    },
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      let param = {}
      Object.assign(param, this.dataForm, {
        page: this.pageIndex,
        limit: this.pageSize
      })
      this.$http({
        url: this.$http.adornUrl('/stock/outbill/list'),
        method: 'get',
        params: this.$http.adornParams(param)
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list
          this.totalPage = data.page.totalCount
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    },
    // 多选
    selectionChangeHandle (val) {
      this.dataListSelections = val
    },
    // 新增 / 修改
    addOrUpdateHandle (outbillId) {
    }
  },
  mounted () {
    this.dataSub = PubSub.subscribe('dataForm', (msg, val) => {
      console.log('~~~~~', val)
      this.dataForm = val
      this.getDataList()
    })
  },
  beforeDestroy () {
    PubSub.unsubscribe(this.dataSub)
  }
}
</script>
