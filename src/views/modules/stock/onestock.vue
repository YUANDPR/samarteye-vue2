<template>

  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form :inline="true" :model="dataForm">
        <el-form-item label="分区">
          <category-cascader :warelocationPath.sync="warelocationPath"></category-cascader>
        </el-form-item>
        <el-form-item label="供用商">
          <brand-select style="width:160px"></brand-select>
        </el-form-item>
        <el-form-item label="保质期">
          <el-input-number v-model="dataForm.qualityPeriod.min" style="width:160px"></el-input-number>
          -
          <el-input-number v-model="dataForm.qualityPeriod.max" style="width:160px"></el-input-number>
        </el-form-item>
        <el-form-item label="检索">
          <el-input v-model="dataForm.key" clearable style="width:160px"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="searchSkuInfo">查询</el-button>
        </el-form-item>
      </el-form>
    </el-form>
    <el-table
      v-loading="dataListLoading"
      :data="dataList"
      border
      style="width: 100%;"
      @selection-change="selectionChangeHandle"
      @expand-change="getSkuDetails"
    >
      <el-table-column type="expand">
        <template slot-scope="scope">
          图片：<img :src="scope.row.onestockImage" style="width:80px;height:80px;"/>
          <br/>
          描述：{{ scope.row.stockDescription }}
          <br/>
          重量：{{ scope.row.weight }}
          <br/>
          分区ID：{{ scope.row.wlId }}
          <br/>
          所属货物ID：{{ scope.row.stockId }}
          <br/>
          供用商ID：{{ scope.row.supplierId }}
          <br/>
          货物种类ID：{{ scope.row.stockTypeId }}
          <br/>
          权限等级：{{ scope.row.levelNumb }}
          <br/>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" type="selection" width="50"></el-table-column>
      <el-table-column align="center" header-align="center" label="onestockId" prop="onestockId"></el-table-column>
      <el-table-column align="center" header-align="center" label="货架" prop="shelfName"></el-table-column>
      <el-table-column align="center" header-align="center" label="货物名称" prop="valueSelect"></el-table-column>
      <el-table-column align="center" header-align="center" label="保质期" prop="qualityPeriod"></el-table-column>
      <el-table-column align="center" header-align="center" label="数量" prop="onestockCount"></el-table-column>
      <el-table-column align="center" header-align="center" label="上架状态" prop="publishStatus">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.publishStatus === 0">新入库</el-tag>
          <el-tag v-if="scope.row.publishStatus === 1">已上架</el-tag>
          <el-tag v-if="scope.row.publishStatus === 2">已下架</el-tag>
        </template>
      </el-table-column>
      <el-table-column align="center" fixed="right" header-align="center" label="操作" width="150">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.publishStatus === 0"
            size="small"
            type="text"
            @click="onestockUp(scope.row.onestockId)"
          >上架
          </el-button>
          <el-dropdown
            size="small"
            split-button
            type="text"
            @command="handleCommand(scope.row,$event)"
          >
            更多
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="stockSettings">库存管理</el-dropdown-item>
              <el-dropdown-item command="levelNumbSettings">叫料权限</el-dropdown-item>
              <el-dropdown-item command="uploadImages">上传图片</el-dropdown-item>
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
  </div>
</template>

<script>
import CategoryCascader from '../common/category-cascader'
import BrandSelect from '../common/supplier-select'
import PubSub from 'pubsub-js'

export default {
  data () {
    return {
      catPathSub: null,
      supplierIdSub: null,
      dataForm: {
        key: '',
        supplierId: undefined,
        wlId: undefined,
        qualityPeriod: {
          min: undefined,
          max: undefined
        }
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      warelocationPath: []
    }
  },
  components: {
    CategoryCascader,
    BrandSelect
  },
  activated () {
    this.getDataList()
  },
  methods: {
    onestockUp (onestockId) {
      this.$http({
        url: this.$http.adornUrl('/stock/onestock/' + onestockId + '/up'),
        method: 'post'
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '上架成功',
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
    },
    getSkuDetails (row, expand) {
      // sku详情查询
      console.log('展开某行...', row, expand)
    },
    // 处理更多指令
    handleCommand (row, command) {
      console.log('~~~~~', row, command)
      if (command === 'stockSettings') {
        this.$router.push({path: '/ware-sku', query: {onestockId: row.onestockId}})
      }
    },
    searchSkuInfo () {
      this.getDataList()
    },
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/stock/onestock/list'),
        method: 'get',
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
          wlId: this.dataForm.wlId,
          supplierId: this.dataForm.supplierId,
          min: this.dataForm.qualityPeriod.min,
          max: this.dataForm.qualityPeriod.max
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
    }
  },
  mounted () {
    this.catPathSub = PubSub.subscribe('catPath', (msg, val) => {
      this.dataForm.wlId = val[val.length - 1]
    })
    this.supplierIdSub = PubSub.subscribe('supplierId', (msg, val) => {
      this.dataForm.supplierId = val
    })
  },
  beforeDestroy () {
    PubSub.unsubscribe(this.catPathSub)
    PubSub.unsubscribe(this.supplierIdSub)
    // eslint-disable-next-line no-irregular-whitespace
  } // 生命周期 - 销毁之前
}
</script>
