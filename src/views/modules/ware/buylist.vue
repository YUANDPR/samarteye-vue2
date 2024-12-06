<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item label="状态">
        <el-select v-model="dataForm.status" clearable placeholder="请选择状态" style="width:120px;">
          <el-option :value="0" label="新建"></el-option>
          <el-option :value="1" label="已分配"></el-option>
          <el-option :value="2" label="已领取"></el-option>
          <el-option :value="3" label="已完成"></el-option>
          <el-option :value="4" label="有异常"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="关键字">
        <el-input v-model="dataForm.key" clearable placeholder="参数名" style="width:120px;"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('ware:purchase:save')"
          type="primary"
          @click="addOrUpdateHandle()"
        >新增
        </el-button>
        <el-button
          v-if="isAuth('ware:purchase:delete')"
          :disabled="dataListSelections.length <= 0"
          type="danger"
          @click="deleteHandle()"
        >批量删除
        </el-button>
      </el-form-item>
    </el-form>
    <el-table
      v-loading="dataListLoading"
      :data="dataList"
      border
      style="width: 100%;"
      @selection-change="selectionChangeHandle"
    >
      <el-table-column align="center" header-align="center" type="selection" width="50"></el-table-column>
      <el-table-column align="center" header-align="center" label="采购单id" prop="buyListId"></el-table-column>
      <el-table-column align="center" header-align="center" label="采购单名称" prop="buyListName"></el-table-column>
      <el-table-column align="center" header-align="center" label="采购人id" prop="assigneeId"></el-table-column>
      <el-table-column align="center" header-align="center" label="采购人名" prop="assigneeName"></el-table-column>
      <el-table-column align="center" header-align="center" label="联系方式" prop="phone"></el-table-column>
      <el-table-column align="center" header-align="center" label="优先级" prop="priority"></el-table-column>
      <el-table-column align="center" header-align="center" label="状态" prop="status">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status == 0">新建</el-tag>
          <el-tag v-if="scope.row.status == 1" type="info">已分配</el-tag>
          <el-tag v-if="scope.row.status == 2" type="warning">已领取</el-tag>
          <el-tag v-if="scope.row.status == 3" type="success">已完成</el-tag>
          <el-tag v-if="scope.row.status == 4" type="danger">有异常</el-tag>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" label="创建日期" prop="createTime"></el-table-column>
      <el-table-column align="center" header-align="center" label="更新日期" prop="updateTime"></el-table-column>
      <el-table-column align="center" fixed="right" header-align="center" label="操作" width="150">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.status==0||scope.row.status==1"
            size="small"
            type="text"
            @click="opendrawer(scope.row)"
          >分配
          </el-button>
          <el-button size="small" type="text" @click="addOrUpdateHandle(scope.row.buyListId)">修改</el-button>
          <el-button size="small" type="text" @click="deleteHandle(scope.row.buyListId)">删除</el-button>
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
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <el-dialog :visible.sync="caigoudialogVisible" title="分配采购人员" width="30%">
      <el-select v-model="eeId" filterable placeholder="请选择">
        <el-option
          v-for="item in eeList"
          :key="item.eeId"
          :label="item.eeName"
          :value="item.eeId"
        ></el-option>
      </el-select>
      <span slot="footer" class="dialog-footer">
        <el-button @click="caigoudialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="assignUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import AddOrUpdate from './buylist-add-or-update'

export default {
  data () {
    return {
      currentRow: {},
      dataForm: {
        key: '',
        status: ''
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      caigoudialogVisible: false,
      eeId: '',
      eeList: []
    }
  },
  components: {
    AddOrUpdate
  },
  activated () {
    this.getDataList()
  },
  created () {

  },
  methods: {
    opendrawer (row) {
      this.geteeList()
      this.currentRow = row
      this.caigoudialogVisible = true
    },
    assignUser () {
      let _this = this
      let user = {}
      this.eeList.forEach(item => {
        if (item.eeId == _this.eeId) {
          user = item
        }
      })
      this.caigoudialogVisible = false
      this.$http({
        url: this.$http.adornUrl(
          `/purchase/buylist/dolist`
        ),
        method: 'post',
        data: this.$http.adornData({
          buyListId: this.currentRow.buyListId || undefined,
          assigneeId: user.eeId,
          assigneeName: user.eeName,
          phone: user.mobile,
          status: 1
        })
      }).then(({data}) => {
        if (data && data.data === 0) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1500
          })

          this.eeId = ''
          this.getDataList()
        } else {
          this.$message({
            message: '操作失败',
            type: 'error',
            duration: 1500
          })
        }
      })
    },
    geteeList () {
      this.$http({
        url: this.$http.adornUrl('/integration/employee/list/eetype'),
        method: 'get',
        params: this.$http.adornParams({
          page: 1,
          limit: 500,
          type: '采购员',
          status: 1
        })
      }).then(({data}) => {
        this.eeList = data.page.list
      })
    },
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/purchase/buylist/list'),
        method: 'get',
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key
        })
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
    addOrUpdateHandle (buyListId) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(buyListId)
      })
    },
    // 删除
    deleteHandle (buyListId) {
      var buyListIds = buyListId
        ? [buyListId]
        : this.dataListSelections.map(item => {
          return item.buyListId
        })
      this.$confirm(
        `确定对[buyListId=${buyListIds.join(',')}]进行[${buyListId ? '删除' : '批量删除'}]操作?`,
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl('/purchase/buylist/delete'),
          method: 'post',
          data: this.$http.adornData(buyListIds, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    }
  }
}
</script>
