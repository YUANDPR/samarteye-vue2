<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" clearable placeholder="员工姓名、职业类型"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <!--   <el-button
             v-if="isAuth('product:brand:save')"
             type="primary"
             @click="addOrUpdateHandle()"
           >新增</el-button>
           <el-button
             v-if="isAuth('product:brand:delete')"
             type="danger"
             @click="deleteHandle()"
             :disabled="dataListSelections.length <= 0"
           >批量删除</el-button>   -->
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
      <el-table-column align="center" header-align="center" label="员工id" prop="eeId"></el-table-column>
      <el-table-column align="center" header-align="center" label="员工名" prop="eeName"></el-table-column>
      <el-table-column align="center" header-align="center" label="手机号码" prop="mobile"></el-table-column>
      <el-table-column align="center" header-align="center" label="邮箱" prop="email"></el-table-column>
      <el-table-column align="center" header-align="center" label="职能类型" prop="jobType"></el-table-column>
      <el-table-column align="center" header-align="center" label="部门id" prop="deptId"></el-table-column>
      <el-table-column align="center" header-align="center" label="叫料权限等级" prop="levelNumb"></el-table-column>
      <el-table-column align="center" header-align="center" label="是否在职" prop="status">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.status"
            :active-value="1"
            :inactive-value="0"
            active-color="#13ce66"
            inactive-color="#ff4949"
            @change="updateBrandStatus(scope.row)"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column align="center" fixed="right" header-align="center" label="操作" width="250">
        <template slot-scope="scope">
          <!--     <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.eeId)">修改</el-button>  -->
          <el-button size="small" type="text" @click="deleteHandle(scope.row.eeId)">删除</el-button>
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

  </div>
</template>

<script>
import AddOrUpdate from './employee-add-or-update'
import CategoryCascader from '../common/category-cascader'

export default {
  data () {
    return {
      dataForm: {
        key: ''
      },
      eeId: 0,
      warelocationPath: [],
      dataList: [],
      cateRelationTableData: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      cateRelationDialogVisible: false,
      popCatelogSelectVisible: false
    }
  },
  components: {
    AddOrUpdate,
    CategoryCascader
  },
  activated () {
    this.getDataList()
  },
  methods: {

    deleteCateRelationHandle (id, eeId) {
      this.$http({
        url: this.$http.adornUrl('/stock/warelocationsupplierrelation/delete'),
        method: 'post',
        data: this.$http.adornData([id], false)
      }).then(({data}) => {
        this.getCateRelation()
      })
    },
    updateCatelogHandle (eeId) {
      this.cateRelationDialogVisible = true
      this.eeId = eeId
      this.getCateRelation()
    },

    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/integration/employee/list'),
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
    updateBrandStatus (data) {
      console.log('最新信息', data)
      let {eeId, status} = data
      //发送请求修改状态
      this.$http({
        url: this.$http.adornUrl('/integration/employee/update/status'),
        method: 'post',
        data: this.$http.adornData({eeId, status}, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '状态更新成功'
        })
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
    addOrUpdateHandle (id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    // 删除
    deleteHandle (id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map(item => {
          return item.eeId
        })
      this.$confirm(
        `确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`,
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl('/integration/employee/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
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
