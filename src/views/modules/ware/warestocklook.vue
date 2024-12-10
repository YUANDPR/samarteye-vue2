<template>
  <div>
    <el-col :span="6">
      <el-card class="background">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">

          <el-form-item label="选择分区">
            <category-cascader :change=getshelf() :warelocationPath.sync="warelocationPath"></category-cascader>

          </el-form-item>
          <el-form-item label="货架">
            <!--  <el-input v-model="dataForm.shelfName" placeholder="货架" clearable></el-input> -->
            <el-col v-for="item in shelf" :key="item.shelfName" :span="12" class="shelf-group">

              <el-button class="button1" type="info" @click="getDataList(item.shelfName)">
                {{ item.shelfName }}
              </el-button>

            </el-col>
          </el-form-item>

        </el-form>
      </el-card>
    </el-col>

    <el-col :span="18">
      <el-card class="background">
        <el-col v-for="item in dataList" :key="item.valueSelect" v-model="dataForm.valueSelect" :span="8"
                class="shelfdata-group">
          <el-button class="shelfdata" type="success">
            {{ item.valueSelect }}
            数量：
            {{ item.realCount }}
          </el-button>
        </el-col>
      </el-card>
    </el-col>
  </div>
</template>

<script>

import AddOrUpdate from './warestock-add-or-update'
import CategoryCascader from '../common/category-cascader'

export default {
  data () {
    return {
      wareList: [],
      dataForm: {
        wlId: '',
        shelfName: ''
      },
      shelf: [],
      warelocationPath: [],
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      firstask: -1,
    }
  },
  components: {
    AddOrUpdate, CategoryCascader

  },
  activated () {  //这里的skuId实际对应shelfName，但是这里是另一个页面链接过来的，我先没改
    console.log('接收到', this.$route.query.skuId)
    if (this.$route.query.skuId) {
      this.dataForm.skuId = this.$route.query.skuId
    }

    //this.getDataList();
    this.getinitshelf()
  },
  methods: {
    // 获取货架
    getshelf () {
      if (this.firstask == this.warelocationPath[2]) {
        return
      }
      this.dataForm.wlId = this.warelocationPath[2]
      this.$http({
        url: this.$http.adornUrl('/stock/shelf/list/' + this.dataForm.wlId),
        method: 'get',
        params: this.$http.adornParams({
          page: 1,
          limit: 100,
        })
      }).then(({data}) => {
        this.shelf = data.page.list
      })
      this.firstask = this.warelocationPath[2]

    },
    //首次加载显示的货架
    getinitshelf () {
      this.$http({
        url: this.$http.adornUrl('/stock/shelf/list/' + 15),
        method: 'get',
        params: this.$http.adornParams({
          page: 1,
          limit: 100,
        })
      }).then(({data}) => {
        this.shelf = data.page.list
      })
    },

    // 获取数据列表
    getDataList (shelfName) {
      this.dataForm.wlId = this.warelocationPath[2]
      this.$http({
        url: this.$http.adornUrl('/ware/warestock/list'),
        method: 'get',
        params: this.$http.adornParams({
          shelfName: shelfName,
          wlId: this.dataForm.wlId
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list
        } else {
          this.dataList = []
        }
      })
    },

  }
}
</script>

<style lang="scss">
.shelfdata-group {
  padding-top: 25px;
  padding-bottom: 25px;
  padding-left: 38px;


}

.shelfdata {
  min-height: 60px;
  min-width: 300px;
  font-size: 16px;
}

.background {
  background-color: rgb(183, 219, 240);
  height: 800px;
}

.shelf-group {
  padding-top: 20px;
  padding-bottom: 15px;
  padding-left: 15px;


}

.button1 {
  min-height: 60px;
  min-width: 150px;
  font-size: 16px;
}

</style>
