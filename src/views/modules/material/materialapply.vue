
<template>
 <div>
   <el-col :span="6" >
   <div style="height: 700px;">
  <el-steps direction="vertical" :active="active">
    <el-step title="1.设置基本信息"></el-step>
    <el-step title="2.核对物料信息"></el-step>
    <el-step title="3.提交申请" description="已提交的申请可转至[物料申请状态]查看"></el-step>
  </el-steps>
  </div>
   </el-col>

   <el-col :span="16" >
  <el-form :model="dataForm" :rules="rules" ref="dataForm" label-width="100px" class="demo-dataForm">

  
  <el-form-item label="申请名称" prop="mbillName" style="width:500px" >
    <el-input v-model="dataForm.mbillName" ></el-input>
  </el-form-item>
  <el-form-item label="备注" prop="mbillDescription">
    <el-input v-model="dataForm.mbillDescription" style="width:500px" type="textarea"></el-input>
  </el-form-item>
  <el-form-item label="申请分区" prop="mbillDescription">
    <el-cascader :options="options" clearable :props="setting" @change="cascadeChange"></el-cascader>
  </el-form-item>

    
  <el-form-item label="" prop="mbillDescription">
    <el-col :span="24" v-show="dataResp.stepd[0]==1">
    <el-card class="box-card" style="width:80%;margin:20px auto">
    <el-tabs tab-position="left" style="width:98%">
            <el-tab-pane
              :label="group.shelfName"
              v-for="(group,gidx) in dataResp.attrGroups"
              :key="group.shelfId"
            >
              <!-- 遍历属性,每个tab-pane对应一个表单，每个属性是一个表单项  spu.baseAttrs[0] = [{materialTypeId:xx,val:}]-->
              <el-form ref="form" :model="spu">
                <el-form-item
                  :label="attr.materialTypeName"
                  v-for="(attr,aidx) in group.materialTypeEntities"
                  :key="attr.materialTypeId"
                >
                  <el-input
                    v-model="dataResp.baseAttrs[gidx][aidx].materialTypeId"
                    type="hidden"
                    v-show="false"
                  ></el-input>
                 
                  <el-select
                    v-model="dataResp.baseAttrs[gidx][aidx].attrValues"
                    :multiple="attr.valueType == 1"
                    filterable
                    allow-create
                    default-first-option
                    placeholder="请选择或输入值"
                  >
                    <el-option
                      v-for="(val,vidx) in attr.valueSelect.split(';')"
                      :key="vidx"
                      :label="val"
                      :value="val"
                    ></el-option>
                  </el-select>
                  数量
                  <el-input size="small" style="width:100px"
                    v-model="dataResp.baseAttrs[gidx][aidx].onematerialCount"
                  ></el-input> 
  
                </el-form-item>
              
              </el-form>
            </el-tab-pane>
          </el-tabs>
         </el-card>
        </el-col>
  </el-form-item>

    <el-form-item>    
       <el-button type="primary" @click="generateSkus" v-show="dataResp.stepd[0]==1">核对物料信息</el-button>         
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
            <el-table-column label="数量" prop="mbilldetailCount">
              <template slot-scope="scope">
                <el-input v-model="scope.row.mbilldetailCount"></el-input>
              </template>
            </el-table-column>
           
            <el-table-column type="expand">
              <template slot-scope="scope">
        
                <el-form :model="scope.row">
                  <el-row>
                    <el-col :span="24">
                      <el-form-item label="设置叫料权限">
                        <el-input-number
                          style="width:160px"
                          :min="0"
                          controls-position="right"
                          v-model="scope.row.levelNumb"
                        ></el-input-number>
                        <label>允许此类员工对该货物进行叫料操作</label>

                      </el-form-item>
                    </el-col>
               
                  </el-row>
                </el-form>
              </template>
            </el-table-column>
          </el-table>
        </el-card>
      </el-col>
  </el-form-item>

  <el-form-item>
    <el-button type="success" @click="submitmaterial" v-show="dataResp.stepd[1]==1">提交申请</el-button>
  </el-form-item>
  
</el-form>
   </el-col>
 </div>
</template>

<<script>
  export default {
    data() {
      return {
        active: 0,
        material: [],
        setting: {
        value: "wlId",
        label: "name",
        children: "children"
        },
        options: [],
        dataForm: {
          wlId: 0,
          mbillName: '',
          mbillDescription: "",
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
          mbillName: [
            { required: true, message: '请输入名称', trigger: 'blur' },
  
          ],
  
          
        }
      };
    },
    methods: {


      getoptions() {
      this.$http({
        url: this.$http.adornUrl("/material/warelocation/list/tree"),
        method: "get"
      }).then(({ data }) => {
        this.options = data.data;
      });
    },
    cascadeChange(path){
       this.$http({
          url: this.$http.adornUrl(
            `/material/shelf/${path[2]}/withMaterialType`
          ),
          method: "get",
          params: this.$http.adornParams({})
        }).then(({ data }) => {
             //先对表单的baseAttrs进行初始化
          if(data.data!=undefined){
            data.data.forEach(item => {
            let attrArray = [];
            if(item.materialTypeEntities!=undefined){
              item.materialTypeEntities.forEach(attr => {
              attrArray.push({
                materialTypeId: attr.materialTypeId,
                attrValues: "",
                onematerialCount: attr.onematerialCount
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
         this.active = 2;
      this.dataResp[1] = 1;
      if (this.dataResp.first) {
      this.dataResp.baseAttrs.forEach(item => {
        item.forEach(attr => {
        this.spu.skus.push({
          materialTypeId: attr.materialTypeId,
          valueSelect: attr.attrValues,
          onematerialCount: attr.onematerialCount,       
          });     
          });
          
      });
      this.$http({
            url: this.$http.adornUrl("/material/onematerial/oneinfo"),
            method: "post",
            data: this.$http.adornData(this.spu.skus, false)
          }).then(({ data }) => {
          //  this.spu.skus = data.data;
          let vos = [];
          data.data.forEach(vo=> {

              vos.push({
                materialTypeId: vo.materialTypeId,
                valueSelect: vo.valueSelect,
                mbilldetailCount: vo.onematerialCount, 
                levelNumb: vo.levelNumb,
                shelfName: vo.shelfName,
                });   
              
                
            });  
            this.spu.skus = vos;
              this.dataResp.stepd[1] = 1;
          });
      }
      this.dataResp.first = false;
      
    },
    submitmaterial(){
         
         this.material = {
                        mbillName: this.dataForm.mbillName,
                        mbillDescription: this.dataForm.mbillDescription,
                        wlId: this.dataForm.wlId,
                        mbilldetialVos: this.spu.skus} 
          console.log("~~~~~", JSON.stringify(this.material));
          this.$http({
            url: this.$http.adornUrl("/material/mbill/save"),
            method: "post",
            data: this.$http.adornData(this.material, false)
          }).then(({ data }) => {
            if (data.code == 0) {
              this.$message({
                type: "success",
                message: "申请已提交!"
              });
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
    },
    created() {
    this.getoptions();
  }
  }
</script>

<style>

</style>