<template>
  <div>
    <el-col :span="6">
      <el-card class="background">

        <el-select v-model="dataForm.wlId" placeholder="请选择仓库">
          <el-option
            v-for="item in wareList"
            :key="item.value"
            :change=getson()
            :label="item.name"
            :value="item.wlId">
          </el-option>
        </el-select>


        <el-col :span="12" class="shelf-group">
          <el-tabs tab-position="left" @tab-click="handleClick">
            <el-tab-pane
              v-for="item in dataList"
              :key="item.wlId"
              :label="item.name"
              :name="item.wlId"
            >
              <el-col v-for="item in dataList2" :key="item.wlId" class="shelf-group">
                <el-button class="button1" type="primary" @click="getupshelf(item.wlId)">
                  {{ item.name }}
                </el-button>
              </el-col>
            </el-tab-pane>

          </el-tabs>
        </el-col>


      </el-card>
    </el-col>

    <el-col :span="18">
      <el-card class="background">
        <el-col v-for="item in shelf" :key="item.shelfName" v-model="dataForm.shelfName" :span="8"
                class="shelfdata-group">
          <el-button class="shelfdata" type="success" @click="getupshelfinfo(item.shelfName)">
            {{ item.shelfName }}
          </el-button>
          <el-progress style="min-width: 300px;" class="custom-progress" :percentage="item.count" :stroke-width="20" :text-inside="true"></el-progress>

        </el-col>
        <el-dialog
          :before-close="handleClose"
          :title="dataInfo[0].shelfName + '|上架的货物信息'"
          :visible.sync="dialogVisible"
          width="45%">
          <span>货架上的货物</span>
          <el-carousel :interval="4000" height="300px" type="card">
            <el-carousel-item v-for="iteminfo in dataInfo" :key="iteminfo.valueSelect">
              <el-card class="card1">
                <el-alert
                  :closable="false"
                  :title="'货物名称：'+iteminfo.valueSelect"
                  effect="dark"
                  type="success">
                </el-alert>
                <el-alert
                  :closable="false"
                  :title="'数量：'+iteminfo.onestockCount"
                  type="success">
                </el-alert>
                <el-alert
                  :closable="false"
                  :title="'保质期：'+iteminfo.qualityPeriod+'天'"
                  show-icon
                  type="warning">
                </el-alert>
                <el-alert
                  :closable="false"
                  :title="'所属货物批次：'+iteminfo.stockId"
                  effect="dark"
                  type="warning">
                </el-alert>
                <el-alert
                  :closable="false"
                  :title="'上架时间：'+iteminfo.updateTime"
                  type="success">
                </el-alert>
              </el-card>
            </el-carousel-item>
          </el-carousel>
          <span slot="footer" class="dialog-footer">
                <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
              </span>
        </el-dialog>

      </el-card>

    </el-col>

  </div>


</template>

<script>
import AddOrUpdate from './warestock-add-or-update'

export default {
  data () {
    return {
      wareList: [],
      dataForm: {
        wlId: '',
        shelfName: ''
      },
      shelf: [],
      dataList: [],
      dataList2: [],
      firstask: '',
      wlIdtemp: 0,
      dataInfo: [{
        shelfName: '',
        qualityPeriod: 0,
        onestockCount: 0,
        valueSelect: '',
        updateTime: '',
        stockId: 0
      }],
      dialogVisible: false

    }
  },
  components: {
    AddOrUpdate

  },
  activated () {  // 这里的skuId实际对应shelfName，但是这里是另一个页面链接过来的，我先没改
    console.log('接收到', this.$route.query.skuId)
    if (this.$route.query.skuId) {
      this.dataForm.skuId = this.$route.query.skuId
    }

    // this.getDataList();
    this.getWares()

    this.getinitson()
  },
  methods: {
    // 获取仓库
    getWares () {
      this.$http({
        url: this.$http.adornUrl('/stock/warelocation/list/son/0'),
        method: 'get'
      }).then(({data}) => {
        this.wareList = data.data
      })
    },
    getson () {
      // eslint-disable-next-line eqeqeq
      if (this.dataForm.wlId == this.firstask) {
        return
      }
      this.$http({
        url: this.$http.adornUrl('/stock/warelocation/list/son/' + this.dataForm.wlId),
        method: 'get'
      }).then(({data}) => {
        this.dataList = data.data
      })
      this.firstask = this.dataForm.wlId
    },

    getinitson () {
      this.$http({
        url: this.$http.adornUrl('/stock/warelocation/list/son/' + 1),
        method: 'get'
      }).then(({data}) => {
        this.dataList = data.data
        this.dataForm.wlId = 1
      })
    },
    // 获取货架
    getshelf (wlId) {
      this.$http({
        url: this.$http.adornUrl('/stock/shelf/list/' + wlId),
        method: 'get'
      }).then(({data}) => {
        this.shelf = data.page.list
      })
    },
    // 获取货架上架情况
    getupshelf (wlId) {
      this.$http({
        url: this.$http.adornUrl('/stock/shelf/list/' + wlId + '/shelfup'),
        method: 'get'
      }).then(({data}) => {
        this.shelf = data.data
      })
      this.wlIdtemp = wlId // 备用， getupshelfinfo会用
    },
    handleClick (tab) {
      let wlpId = tab.name
      this.getson2(wlpId)
    },
    getson2 (wlpId) {
      this.$http({
        url: this.$http.adornUrl('/stock/warelocation/list/son/' + wlpId),
        method: 'get'
      }).then(({data}) => {
        this.dataList2 = data.data
      })
    },

    // 货架上架的详细信息
    getupshelfinfo (shelfName) {
      this.$http({
        url: this.$http.adornUrl('/stock/shelf/list/upshelfinfo'),
        method: 'get',
        params: this.$http.adornParams({
          shelfName: shelfName,
          wlId: this.wlIdtemp
        })
      }).then(({data}) => {
        // eslint-disable-next-line eqeqeq
        if (data.code == 101010) {
          alert(data.msg)
        } else {
          this.dataInfo = data.data
          this.dialogVisible = true
        }
      })
    },
    // 关闭弹窗前的方法
    handleClose () {
      this.dialogVisible = false
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
    }

  }
}
</script>

<style lang="scss">
.shelfdata-group {
  padding: 25px 50px 25px 38px;


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
  padding-left: 8px;

  > .el-tabs {
    padding-top: 20px;
    padding-bottom: 15px;
    width: 350px;
    left: 0;

  }

}

.button1 {

  min-height: 60px;
  min-width: 150px;
  font-size: 16px;
}

.card1 {
  background-color: rgb(110, 231, 225);
}
</style>

<style scoped>
.custom-progress >>> .el-progress-bar__innerText {
  color: #000000;
}
</style>
