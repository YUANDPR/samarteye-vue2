
<template>
 <div>
   <el-col :span="6" >
   <div style="height: 700px;">
  <el-steps direction="vertical" :active="active">
    <el-step title="1.设置基本信息"></el-step>
    <el-step title="2.核对库存信息"></el-step>
    <el-step title="3.生成出库单" description="已生成的出库单可转至[出库单]查看"></el-step>
  </el-steps>
  </div>
   </el-col>

   <el-col :span="16" >
  <el-form :model="dataForm" :rules="rules" ref="dataForm" label-width="100px" class="demo-dataForm">

  
  <el-form-item label="出库类型" prop="outtype" style="width:500px" >
  <!--  <el-input v-model="dataForm.mbillName" ></el-input> -->
    <div style="margin-top: 30px">
    <el-radio-group v-model="dataForm.outtype" size="medium">
      <el-radio-button label="物料申请单出库" ></el-radio-button>
      <el-radio-button label="手动出库"></el-radio-button>
      <el-radio-button label="其他出库"></el-radio-button>
    </el-radio-group>
  </div>
  </el-form-item>
  <el-form-item label="物料申请单ID" prop="mbillId" v-show="dataForm.outtype=='物料申请单出库'">
    <el-input v-model="dataForm.mbillId" :value="dataForm.mbillId" style="width:150px"></el-input>
  </el-form-item>
   <el-form-item label="出库单名称" prop="outbillDescription">
    <el-input v-model="dataForm.outbillName" style="width:250px" ></el-input>
  </el-form-item>
  <el-form-item label="备注" prop="outbillDescription">
    <el-input v-model="dataForm.outbillDescription" style="width:500px" type="textarea"></el-input>
  </el-form-item>
  <el-form-item label="出库分区" >
    <el-cascader :options="options" clearable :props="setting" @change="cascadeChange"></el-cascader>
  </el-form-item>

    
  <el-form-item label="" prop="mbillDescription"  v-show="dataResp.stepd[0]==1" >
    <el-col :span="15">
    <el-card class="box-card" style="width:95%;margin:20px auto">
    <el-tabs tab-position="left" style="width:98%">
            <el-tab-pane
              :label="group.shelfName"
              v-for="(group,gidx) in dataResp.attrGroups"
              :key="group.shelfId"
            >
              <!-- 遍历属性,每个tab-pane对应一个表单，每个属性是一个表单项  spu.baseAttrs[0] = [{stockTypeId:xx,val:}]-->
              <el-form ref="form" :model="spu">
                <el-form-item
                  :label="attr.stockTypeName"
                  v-for="(attr,aidx) in group.stockTypeEntities"
                  :key="attr.stockTypeId"
                >
                  <el-input
                    v-model="dataResp.baseAttrs[gidx][aidx].stockTypeId"
                    type="hidden"
                    v-show="false"
                  ></el-input>
                 
                  <el-select
                    v-model="dataResp.baseAttrs[gidx][aidx].attrValues"
                    :multiple="attr.valueType == 1"
                    filterable
                    allow-create
                    default-first-option
                    @change="selectCount(group.shelfName,dataResp.baseAttrs[gidx][aidx].attrValues)"
                    placeholder="请选择或输入值"
                    style="width:150px"
                  >
                    <el-option
                      v-for="(val,vidx) in attr.valueSelect.split(';')"
                      :key="vidx"
                      :label="val"
                      :value="val"
                    ></el-option>
                  </el-select>
                
                  出库数量
                  <el-input size="small" style="width:60px"
                    v-model="dataResp.baseAttrs[gidx][aidx].onestockCount"
                  ></el-input> 
  
                </el-form-item>
              
              </el-form>
            </el-tab-pane>
          </el-tabs>
         </el-card>
        </el-col>
          <el-col :span="5" >
                  库存剩余
                 <el-input size="small" style="width:115px" disabled 
                    v-model="realCount"
                  ></el-input> 
                </el-col>
  </el-form-item>

    <el-form-item >    
       <el-button type="primary" @click="generateSkus" v-show="dataResp.stepd[0]==1">核对库存信息</el-button>         
   </el-form-item>

  <el-form-item>
  <el-col :span="24" v-show="dataResp.stepd[1]==1">
        <el-card class="box-card" style="width:80%;margin:20px auto">
          <el-table :data="spu.skus" style="width: 100%">
            <el-table-column label="货架名称">
               <template slot-scope="scope">
                <el-input v-model="scope.row.shelfName"></el-input>
              </template>
      
            </el-table-column>
            <el-table-column label="货物名称" prop="valueSelect">
              <template slot-scope="scope">
                <el-input v-model="scope.row.valueSelect"></el-input>
              </template>
            </el-table-column>
            <el-table-column label="数量" prop="outbilldetailCount">
              <template slot-scope="scope">
                <el-input v-model="scope.row.outbilldetailCount"></el-input>
              </template>
            </el-table-column>
           
          </el-table>
        </el-card>
      </el-col>
  </el-form-item>

  <el-form-item>
    <el-button type="success" @click="submitstock" v-show="dataResp.stepd[1]==1">生成出库单</el-button>
  </el-form-item>
  
</el-form>
   </el-col>
 </div>
</template>

<<script>
  export default {
    data() {
      return {
        mitoken: "",
        realCount: '',
        active: 0,
        outbill: [],
        setting: {
        value: "wlId",
        label: "name",
        children: "children"
        },
        options: [],
        dataForm: {
          
          outbillName: "",  
          wlId: 0,
          mbillId: 0,
          outtype: '手动出库',
          outtypeint: 0,
          outbillDescription: "",
        },
        spu: {
        skus: [] 
      },
       dataResp: {
        attrGroups: [],
         baseAttrs: [],
         stepd: [0,0],
         first: true
      },
        rules: {
          outtype: [
            { required: true, message: '请输入名称', trigger: 'blur' },
  
          ],
  
          
        }
      };
    },
    activated() {
      if (this.$route.query.mbillId) {
      this.dataForm.mbillId = this.$route.query.mbillId;
      this.dataForm.outtype = "物料申请单出库";
      }
    },
    methods: {
      selectCount(shelfName,valueSelect){
        this.$http({
            url: this.$http.adornUrl("/ware/warestock/count"),
            method: "get",
            params: this.$http.adornParams({shelfName:shelfName,valueSelect:valueSelect,wlId:this.dataForm.wlId})
          }).then(({ data }) => {
              this.realCount = data.data;
              if (data.data == -1) {
                this.realCount = '不存在该库存';
              }
    

           });
      },

      getoptions() {
      this.$http({
        url: this.$http.adornUrl("/stock/warelocation/list/tree"),
        method: "get"
      }).then(({ data }) => {
        this.options = data.data;
      });
    },
    cascadeChange(path){
       this.$http({
          url: this.$http.adornUrl(
            `/stock/shelf/${path[2]}/withStockType`
          ),
          method: "get",
          params: this.$http.adornParams({})
        }).then(({ data }) => {
             //先对表单的baseAttrs进行初始化
          if(data.data!=undefined){
            data.data.forEach(item => {
            let attrArray = [];
            if(item.stockTypeEntities!=undefined){
              item.stockTypeEntities.forEach(attr => {
              attrArray.push({
                stockTypeId: attr.stockTypeId,
                attrValues: "",
                onestockCount: attr.onestockCount
              });
            });
            }
            this.dataResp.baseAttrs.push(attrArray);
          });
          }
          this.dataResp.stepd[0] = 1;
          this.dataResp.attrGroups = data.data;
          this.dataForm.wlId = path[2]
            });
      },
      generateSkus() {
         this.active = 1;
      this.dataResp[1] = 1;
      if (this.dataResp.first) {
      this.dataResp.baseAttrs.forEach(item => {
        item.forEach(attr => {
        this.spu.skus.push({
          stockTypeId: attr.stockTypeId,
          valueSelect: attr.attrValues,
          onestockCount: attr.onestockCount,       
          });     
          });
          
      });
      this.$http({
            url: this.$http.adornUrl("/stock/onestock/oneinfo"),
            method: "post",
            data: this.$http.adornData(this.spu.skus, false)
          }).then(({ data }) => {
          //  this.spu.skus = data.data;
          let vos = [];
          data.data.forEach(vo=> {

              vos.push({
                stockTypeId: vo.stockTypeId,
                valueSelect: vo.valueSelect,
                outbilldetailCount: vo.onestockCount,
                shelfName: vo.shelfName,
                });   
              
                
            });  
            this.spu.skus = vos;
              this.dataResp.stepd[1] = 1;
          });
      }
      this.dataResp.first = false;
      
    },
    submitstock(){
         if ("物料申请单出库"==this.dataForm.outtype) {
             this.dataForm.outtypeint = 1;
         }
         if ("手动出库"==this.dataForm.outtype) {
             this.dataForm.outtypeint = 2;
         }
         if ("其他出库"==this.dataForm.outtype) {
             this.dataForm.outtypeint = 3;
         }
        
         this.outbill = {
                        outbillName: this.dataForm.outbillName,
                        outtype: this.dataForm.outtypeint,
                        outbillDescription: this.dataForm.outbillDescription,
                        wlId: this.dataForm.wlId,
                        mbillId: this.dataForm.mbillId,
                        outbilldetialVos: this.spu.skus,
                        miToken:this.mitoken} 
          console.log("~~~~~", JSON.stringify(this.outbill));
          this.$http({
            url: this.$http.adornUrl("/stock/outbill/save"),
            method: "post",
            data: this.$http.adornData(this.outbill, false)
          }).then(({ data }) => {
            if (data.code == 0) {
              this.$message({
                type: "success",
                message: "申请已提交!"
              });
            if ("物料申请单出库"==this.dataForm.outtype) {
              this.$http({
              url: this.$http.adornUrl("/material/mbill/" + this.dataForm.mbillId + "/handle/1"),
              method: "post"
              }).then(({ data }) => {});
            }
              location.reload();
            } else {
              this.$message({
                type: "error",
                message: "申请失败，原因【" + data.msg + "】"
              });
            }
              this.active = 2;
          });
          
    },
    getToken(){
      this.$http({
          url: this.$http.adornUrl(
            `/stock/outbill/getToken`
          ),
          method: "get",
        }).then(({ data }) => {
          if(data.code==0){
            this.mitoken = data.data
          }
            
        })
    }
    },

    created() {
    this.getoptions();
    this.getToken();

  }
  }
</script>

<style>

</style>