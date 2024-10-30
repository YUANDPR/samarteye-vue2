<template>
    <div > 
      <el-col :span="6">
        <el-card class="backgroud">

        <el-select v-model="dataForm.wlId" placeholder="请选择仓库">
            <el-option
            v-for="item in wareList"
            :key="item.value"
            :label="item.name"
            :value="item.wlId"
            :change=getson()>
            </el-option>
        </el-select>


      <!--  <el-input v-model="dataForm.shelfName" placeholder="货架" clearable></el-input> -->
        <el-col :span="12" class="shelf-group"  >
           
     <!--     <el-button @click="getshelf(item.wlId)" class="button1" type="info"> 
            {{ item.name }}
            </el-button> -->
             <el-tabs tab-position="left"  @tab-click="handleClick" 
              >
              <el-tab-pane
               v-for="item in dataList" 
               :key="item.wlId"
              :label="item.name"
              :name="item.wlId"
              >
              <el-col v-for="item in dataList2"  :key="item.wlId" class="shelf-group">
              <el-button @click="getupshelf(item.wlId)" class="button1" type="primary" > 
            {{ item.name }}
            </el-button>
             </el-col>
               </el-tab-pane>
              
            </el-tabs>
        </el-col>


    
        </el-card>
      </el-col>

      <el-col :span="18">
        <el-card class="backgroud">
         <el-col :span="8" class="shelfdata-group"  v-for="item in shelf" :key="item.shelfName" v-model="dataForm.shelfName"> 
           <el-button type="success" class="shelfdata"  @click="getupshelfinfo(item.shelfName)" >
            {{item.shelfName}}
           </el-button>
           <el-progress :text-inside="true" :stroke-width="20" :percentage="item.count*5"  ></el-progress>
      
        </el-col>
             <el-dialog 
              :title="dataInfo[0].shelfName + '|上架的货物信息'"
              :visible.sync="dialogVisible"
              width="45%"
              :before-close="handleClose">
              <span>货架上的货物</span>
               <el-carousel :interval="4000" type="card" height="300px">
                <el-carousel-item v-for="iteminfo in dataInfo" :key="iteminfo.valueSelect">
                  <el-card class="card1">
                  <el-alert
                    :title="'货物名称：'+iteminfo.valueSelect"
                    type="success"
                    :closable="false"
                    effect="dark">
                  </el-alert>
                   <el-alert
                   :title="'数量：'+iteminfo.onestockCount"
                    type="success"
                    :closable="false">
                  </el-alert>
                  <el-alert
                    :title="'保质期：'+iteminfo.qualityPeriod+'天'"
                    type="warning"
                    :closable="false"
                    show-icon>
                  </el-alert>
                  <el-alert
                    :title="'所属货物批次：'+iteminfo.stockId"
                    type="warning"
                    :closable="false"
                    effect="dark">
                  </el-alert>
                  <el-alert
                    :title="'上架时间：'+iteminfo.updateTime"
                    :closable="false"
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

import AddOrUpdate from "./warestock-add-or-update";

export default {
  data() {
    return {
      wareList: [],
      dataForm: {
        wlId:"",
        shelfName: ""
      },
      shelf:[],
      dataList: [],
      dataList2:[],
      firstask:"",
      wlIdtemp:0,
      dataInfo:[{
        shelfName:"",
        qualityPeriod:0,
        onestockCount:0,
        valueSelect:"",
        updateTime:"",
        stockId:0}],
      dialogVisible: false,

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
  
   //this.getDataList();
     this.getWares();
     
    this.getinitson();

  },
  methods: {
    //获取仓库
      getWares() {  
      this.$http({
        url: this.$http.adornUrl("/stock/warelocation/list/son/0"),
        method: "get",
      }).then(({ data }) => {
        this.wareList = data.data;
      });
    },   
    getson(){
        if(this.dataForm.wlId==this.firstask){
            return;
        }
        this.$http({
        url: this.$http.adornUrl("/stock/warelocation/list/son/"+this.dataForm.wlId),
        method: "get",
      }).then(({ data }) => {
        this.dataList = data.data;
      });
      this.firstask = this.dataForm.wlId;
    },
    
    getinitson(){
       this.$http({
        url: this.$http.adornUrl("/stock/warelocation/list/son/"+1),
        method: "get",
      }).then(({ data }) => {
        this.dataList = data.data;
        this.dataForm.wlId = 1;
      });
    },
    // 获取货架
    getshelf(wlId){ 
        this.$http({
        url: this.$http.adornUrl("/stock/shelf/list/"+wlId),
        method: "get",
      }).then(({ data }) => {
        this.shelf = data.page.list;
      });
    }, 
    // 获取货架上架情况
    getupshelf(wlId){ 
        this.$http({
        url: this.$http.adornUrl("/stock/shelf/list/"+wlId+"/shelfup"),
        method: "get",
      }).then(({ data }) => {
        this.shelf = data.data;
      });
      this.wlIdtemp = wlId; //备用， getupshelfinfo会用
    }, 
     handleClick(tab) {
        
        let wlpId = tab.name;
        this.getson2(wlpId);
        
      },
      getson2(wlpId){
      this.$http({
        url: this.$http.adornUrl("/stock/warelocation/list/son/"+wlpId),
        method: "get",
      }).then(({ data }) => {
        this.dataList2 = data.data;
      });
      },

    //货架上架的详细信息
    getupshelfinfo(shelfName){
      this.$http({
        url: this.$http.adornUrl("/stock/shelf/list/upshelfinfo"),
        method: "get",
         params: this.$http.adornParams({
          shelfName: shelfName,
          wlId: this.wlIdtemp
        })
      }).then(({ data }) => {
        if(data.code==101010){
          alert(data.msg);
        }else{
          this.dataInfo = data.data;
          this.dialogVisible = true;
        }

      });
      
  
    },
    //关闭弹窗前的方法
    handleClose(done) {
        
    },


    // 获取数据列表
    getDataList(shelfName) {
      this.dataForm.wlId = this.warelocationPath[2];
      this.$http({
        url: this.$http.adornUrl("/ware/warestock/list"),
        method: "get",
        params: this.$http.adornParams({
          shelfName: shelfName,
          wlId: this.dataForm.wlId
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list;
        } else {
          this.dataList = [];
        }
      });
    },

  }
};
</script>

<style lang="scss">
.shelfdata-group{
    padding-top: 25px;
    padding-bottom: 25px;
    padding-left: 38px;
    padding-right: 50px;


}
.shelfdata{
   min-height: 60px;
   min-width: 300px;
   font-size: 16px;
}
.backgroud{
    background-color: rgb(183, 219, 240);
     height: 800px;
}
.shelf-group{
    padding-top: 20px;
    padding-bottom: 15px;
    padding-left: 8px;
    >.el-tabs{
       padding-top: 20px;
       padding-bottom: 15px;
      width: 350px;
      left: 0;

    }
    
}
.button1{

     min-height: 60px;
     min-width: 150px;
     font-size: 16px;
}

.card1{
   background-color: rgb(110, 231, 225);
}
</style>