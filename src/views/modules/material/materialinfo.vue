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
      <el-table-column align="center" header-align="center" label="materialId" prop="materialId"></el-table-column>
      <el-table-column align="center" header-align="center" label="货物名称" prop="materialName"></el-table-column>
      <el-table-column align="center" header-align="center" label="描述" prop="materialDescription"></el-table-column>
      <el-table-column align="center" header-align="center" label="分类" prop="wlId"></el-table-column>

      <el-table-column align="center" header-align="center" label="上架状态" prop="publishStatus">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.publishStatus == 0">新建</el-tag>
          <el-tag v-if="scope.row.publishStatus == 1">已上架</el-tag>
          <el-tag v-if="scope.row.publishStatus == 2">已下架</el-tag>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" label="创建时间" prop="createTime"></el-table-column>
      <el-table-column align="center" header-align="center" label="修改时间" prop="updateTime"></el-table-column>
      <el-table-column align="center" fixed="right" header-align="center" label="操作" width="150">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.publishStatus == 0"
            size="small"
            type="text"
            @click="materialUp(scope.row.materialId)"
          >上架
          </el-button>
          <el-button size="small" type="text" @click="detailShow(scope.row.materialId)">详情</el-button>
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
        <el-table-column label="数量" property="onematerialCount"></el-table-column>

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
        onematerialCount: 0,
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
    materialUp (materialId) {
      this.$http({
        url: this.$http.adornUrl('/material/material/' + materialId + '/up'),
        method: 'post'
      }).then(({data}) => {
        if (data && data.data === 1) {
          this.$message({
            message: '上架成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.getDataList()
            }
          })
        } else {
          this.$message.error('该(批)货物中存在未上架的货物单元，请先上架它们')
        }
      })
    },
    detailShow (materialId) {
      this.$http({
        url: this.$http.adornUrl(`/material/onematerial/${materialId}/info`),
        method: 'get',
      }).then(({data}) => {
        this.gridData = data.data
      })
      this.dialogTableVisible = true

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
        url: this.$http.adornUrl('/material/material/list'),
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
    addOrUpdateHandle (materialId) {
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
