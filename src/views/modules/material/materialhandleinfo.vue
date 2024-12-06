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
      <el-table-column align="center" header-align="center" label="mbillId" prop="mbillId"></el-table-column>
      <el-table-column align="center" header-align="center" label="申请单名称" prop="mbillName"></el-table-column>
      <el-table-column align="center" header-align="center" label="描述" prop="mbillDescription"></el-table-column>
      <el-table-column align="center" header-align="center" label="分区编号" prop="wlId"></el-table-column>

      <el-table-column align="center" header-align="center" label="申请状态" prop="status">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status == 0">申请中</el-tag>
          <el-tag v-if="scope.row.status == 1" type="info">处理中</el-tag>
          <el-tag v-if="scope.row.status == 2" type="success">已完成</el-tag>
          <el-tag v-if="scope.row.status == 3" type="danger">拒绝申请</el-tag>
          <el-tag v-if="scope.row.status == 4" type="warning">撤回申请</el-tag>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" label="申请时间" prop="createTime"></el-table-column>
      <el-table-column align="center" header-align="center" label="更新时间" prop="updateTime"></el-table-column>
      <el-table-column align="center" fixed="right" header-align="center" label="操作" width="150">
        <template slot-scope="scope">

          <el-button size="small" type="text" @click="detailShow(scope.row.mbillId)">详情</el-button>
          <el-dropdown
            size="small"
            split-button
            type="text"
            @command="handleCommand(scope.row.mbillId,scope.row.status,$event)"
          >
            处理申请
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="handle">处理并生成出库单</el-dropdown-item>
              <el-dropdown-item command="refuse">拒绝申请</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
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
    <el-dialog :visible.sync="dialogTableVisible" title="物料申请详情">
      <el-table :data="gridData">
        <el-table-column label="货架" property="shelfName" width="150"></el-table-column>
        <el-table-column label="名称" property="valueSelect" width="200"></el-table-column>
        <el-table-column label="数量" property="mbilldetailCount"></el-table-column>

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
        mbilldetailCount: 0,
        shelfName: ''
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
    //处理申请指令
    handleCommand (mbillId, status, command) {

      if (command === 'handle') {
        if (status === 0) {
          this.$router.push({path: '/stock-outbilladd', query: {mbillId: mbillId}})
        } else {
          this.$message.error('该申请已存在出库单，不允许重复创建')
        }

      }
      if (command === 'refuse') {
        this.$http({
          url: this.$http.adornUrl('/material/mbill/' + mbillId + '/handle/3'),
          method: 'post'
        }).then(({data}) => {
          if (data && data.data === 1) {
            this.$message({
              message: '申请已拒绝',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error('申请已在处理中，无法拒绝')
          }
        })
      }
    },

    materialUp (mbillId) {
      this.$http({
        url: this.$http.adornUrl('/material/mbill/' + mbillId + '/revoke'),
        method: 'post'
      }).then(({data}) => {
        if (data && data.data === 1) {
          this.$message({
            message: '申请已撤销',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.getDataList()
            }
          })
        } else {
          this.$message.error('申请已在处理中，无法撤回，请联系库存管理员')
        }
      })
    },
    detailShow (mbillId) {
      this.$http({
        url: this.$http.adornUrl(`/material/mbilldetail/${mbillId}/info`),
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
        url: this.$http.adornUrl('/material/mbill/list'),
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
    addOrUpdateHandle (mbillId) {
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
