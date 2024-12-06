<template>
  <div>
    <el-row>
      <el-col :span="24">
        <el-steps :active="step" finish-status="success">
          <el-step title="物料信息"></el-step>
          <el-step title="物料基本信息"></el-step>
          <el-step title="货架外信息"></el-step>
          <el-step title="其他信息"></el-step>
          <el-step title="保存完成"></el-step>
        </el-steps>
      </el-col>
      <el-col v-show="step==0" :span="24">
        <el-card class="box-card" style="width:80%;margin:20px auto">
          <el-form ref="spuBaseForm" :model="spu" :rules="spuBaseInfoRules" label-width="120px">
            <el-form-item label="物料名称" prop="spuName">
              <el-input v-model="spu.spuName"></el-input>
            </el-form-item>
            <el-form-item label="物料描述" prop="spuDescription">
              <el-input v-model="spu.spuDescription"></el-input>
            </el-form-item>
            <el-form-item label="选择分区" prop="wlId">
              <category-cascader></category-cascader>
            </el-form-item>
            <el-form-item label="物料重量(Kg)" prop="weight">
              <el-input-number v-model.number="spu.weight" :min="0" :precision="3" :step="0.1"></el-input-number>
            </el-form-item>
            <!--  <el-form-item label="设置积分" prop="bounds">
                <label>金币</label>
                <el-input-number
                  style="width:130px"
                  placeholder="金币"
                  v-model="spu.bounds.buyBounds"
                  :min="0"
                  controls-position="right"
                ></el-input-number>
                <label style="margin-left:15px">成长值</label>
                <el-input-number
                  style="width:130px"
                  placeholder="成长值"
                  v-model="spu.bounds.growBounds"
                  :min="0"
                  controls-position="right"
                >
                  <template slot="prepend">成长值</template>
                </el-input-number>
              </el-form-item> -->
            <el-form-item label="物料简图" prop="decript">
              <multi-upload v-model="spu.decript"></multi-upload>
            </el-form-item>

            <el-form-item label="物料详细图集" prop="images">
              <multi-upload v-model="spu.images"></multi-upload>
            </el-form-item>
            <el-form-item>
              <el-button type="success" @click="collectSpuBaseInfo">下一步：设置物料基本信息</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
      <el-col v-show="step==1" :span="24">
        <el-card class="box-card" style="width:80%;margin:20px auto">
          <el-tabs style="width:98%" tab-position="left">
            <el-tab-pane
              v-for="(group,gidx) in dataResp.attrGroups"
              :key="group.shelfId"
              :label="group.shelfName"
            >
              <!-- 遍历属性,每个tab-pane对应一个表单，每个属性是一个表单项  spu.baseAttrs[0] = [{materialTypeId:xx,val:}]-->
              <el-form ref="form" :model="spu">
                <el-form-item
                  v-for="(attr,aidx) in group.materialTypeEntities"
                  :key="attr.materialTypeId"
                  :label="attr.materialTypeName"
                >
                  <el-input
                    v-show="false"
                    v-model="dataResp.baseAttrs[gidx][aidx].materialTypeId"
                    type="hidden"
                  ></el-input>

                  <el-select
                    v-model="dataResp.baseAttrs[gidx][aidx].attrValues"
                    :multiple="attr.valueType == 1"
                    allow-create
                    default-first-option
                    filterable
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
                  <el-input v-model="dataResp.baseAttrs[gidx][aidx].onematerialCount" size="small"
                            style="width:100px"
                  ></el-input>

                </el-form-item>
              </el-form>
            </el-tab-pane>
          </el-tabs>
          <div style="margin:auto">
            <el-button type="primary" @click="step = 0">上一步</el-button>
            <el-button type="success" @click="generateSaleAttrs">下一步：设置货架外信息</el-button>
          </div>
        </el-card>
      </el-col>
      <el-col v-show="step==2" :span="24">
        <el-card class="box-card" style="width:80%;margin:20px auto">
          <el-card class="box-card">
            <div slot="header" class="clearfix">
              <span>存放在货架之外的物料</span>
              <el-form ref="saleform" :model="spu">
                <el-form-item
                  v-for="(attr,aidx) in dataResp.saleAttrs"
                  :key="attr.materialTypeId"
                  :label="attr.materialTypeName"
                >
                  <el-input
                    v-show="false"
                    v-model="dataResp.tempSaleAttrs[aidx].materialTypeId"
                    type="hidden"
                  ></el-input>
                  <el-checkbox-group v-model="dataResp.tempSaleAttrs[aidx].attrValues">
                    <el-checkbox
                      v-for="val in dataResp.saleAttrs[aidx].valueSelect.split(';')"
                      v-if="dataResp.saleAttrs[aidx].valueSelect != ''"
                      :key="val"
                      :label="val"
                    ></el-checkbox>
                    <div style="margin-left:20px;display:inline">
                      <el-button
                        v-show="!inputVisible[aidx].view"
                        class="button-new-tag"
                        size="mini"
                        @click="showInput(aidx)"
                      >+自定义
                      </el-button>
                      <el-input
                        v-show="inputVisible[aidx].view"
                        :ref="'saveTagInput'+aidx"
                        v-model="inputValue[aidx].val"
                        size="mini"
                        style="width:150px"
                        @blur="handleInputConfirm(aidx)"
                        @keyup.enter.native="handleInputConfirm(aidx)"
                      ></el-input>
                    </div>
                  </el-checkbox-group>
                </el-form-item>
              </el-form>
            </div>
            <el-button type="primary" @click="step = 1">上一步</el-button>
            <el-button type="success" @click="generateSkus">下一步：设置其他信息</el-button>
          </el-card>
        </el-card>
      </el-col>
      <el-col v-show="step==3" :span="24">
        <el-card class="box-card" style="width:80%;margin:20px auto">
          <el-table :data="spu.skus" style="width: 100%">
            <el-table-column label="货架名称">
              <template slot-scope="scope">
                <el-input v-model="scope.row.shelfName"></el-input>
              </template>
              <!--     <el-table-column
                     :label="item.materialTypeName"
                     v-for="(item,index) in dataResp.tableAttrColumn"
                     :key="item.materialTypeId"
                   >
                     <template slot-scope="scope">
                       <span style="margin-left: 10px">{{ scope.row.attr[index].attrValue }}</span>
                     </template>
                   </el-table-column> -->
            </el-table-column>
            <el-table-column label="物料名称" prop="valueSelect">
              <template slot-scope="scope">
                <el-input v-model="scope.row.valueSelect"></el-input>
              </template>
            </el-table-column>
            <el-table-column label="数量" prop="onematerialCount">
              <template slot-scope="scope">
                <el-input v-model="scope.row.onematerialCount"></el-input>
              </template>
            </el-table-column>
            <el-table-column label="保质期" prop="qualityPeriod">
              <template slot-scope="scope">
                <el-input v-model="scope.row.qualityPeriod" type="number"></el-input>
              </template>
            </el-table-column>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!--      <el-row>
                        <el-col :span="24">
                          <label style="display:block;float:left">选择图集 或</label>
                          <multi-upload
                            style="float:left;margin-left:10px;"
                            :showFile="false"
                            :listType="'text'"
                            v-model="uploadImages"
                          ></multi-upload>
                        </el-col>
                        <el-col :span="24">
                          <el-divider></el-divider>
                        </el-col>
                        <el-col :span="24">
                          <el-card
                            style="width:170px;float:left;margin-left:15px;margin-top:15px;"
                            :body-style="{ padding: '0px' }"
                            v-for="(img,index) in spu.images"
                            :key="index"
                          >
                            <img :src="img" style="width:160px;height:120px" />
                            <div style="padding: 14px;">
                              <el-row>
                                <el-col :span="12">
                                  <el-checkbox
                                    v-model="scope.row.images[index].imgUrl"
                                    :true-label="img"
                                    false-label
                                  ></el-checkbox>
                                </el-col>
                                <el-col :span="12">
                                  <el-tag v-if="scope.row.images[index].defaultImg == 1">
                                    <input
                                      type="radio"
                                      checked
                                      :name="scope.row.skuName"
                                      @change="checkDefaultImg(scope.row,index,img)"
                                    />设为默认
                                  </el-tag>
                                  <el-tag v-else>
                                    <input
                                      type="radio"
                                      :name="scope.row.skuName"
                                      @change="checkDefaultImg(scope.row,index,img)"
                                    />设为默认
                                  </el-tag>
                                </el-col>
                              </el-row>
                            </div>
                          </el-card>
                        </el-col>
                      </el-row>  -->
                <!-- 折扣，满减，会员价 -->
                <el-form :model="scope.row">
                  <el-row>
                    <el-col :span="24">
                      <el-form-item label="设置叫料权限">
                        <el-input-number
                          v-model="scope.row.levelNumb"
                          :min="0"
                          controls-position="right"
                          style="width:160px"
                        ></el-input-number>
                        <label>允许此类员工对该物料进行叫料操作</label>

                        <!--       <label style="margin-left:15px;">打</label>
                               <el-input-number
                                 style="width:160px"
                                 v-model="scope.row.disonematerialCount"
                                 :precision="2"
                                 :max="1"
                                 :min="0"
                                 :step="0.01"
                                 controls-position="right"
                               ></el-input-number>
                               <label>折</label>
                               <el-checkbox
                                 v-model="scope.row.onematerialCountStatus"
                                 :true-label="1"
                                 :false-label="0"
                               >可叠加优惠</el-checkbox> -->
                      </el-form-item>
                    </el-col>
                    <!--  <el-col :span="24">
                        <el-form-item label="设置满减">
                          <label>满</label>
                          <el-input-number
                            style="width:160px"
                            v-model="scope.row.fullPrice"
                            :step="100"
                            :min="0"
                            controls-position="right"
                          ></el-input-number>
                          <label>元</label>
                          <label style="margin-left:15px;">减</label>
                          <el-input-number
                            style="width:160px"
                            v-model="scope.row.reducePrice"
                            :step="10"
                            :min="0"
                            controls-position="right"
                          ></el-input-number>
                          <label>元</label>
                          <el-checkbox
                            v-model="scope.row.priceStatus"
                            :true-label="1"
                            :false-label="0"
                          >可叠加优惠</el-checkbox>
                        </el-form-item>
                      </el-col> -->

                    <!--    <el-col :span="24">
                             <el-form-item label="设置会员价" v-if="scope.row.memberPrice.length>0">
                               <br />

                               <el-form-item v-for="(mp,mpidx) in scope.row.memberPrice" :key="mp.id">
                                 {{mp.name}}
                                 <el-input-number
                                   style="width:160px"
                                   v-model="scope.row.memberPrice[mpidx].price"
                                   :precision="2"
                                   :min="0"
                                   controls-position="right"
                                 ></el-input-number>
                               </el-form-item>
                             </el-form-item>
                           </el-col>  -->
                  </el-row>
                </el-form>
              </template>
            </el-table-column>
          </el-table>
          <el-button type="primary" @click="step = 2">上一步</el-button>
          <el-button type="success" @click="submitSkus">下一步：完成入库</el-button>
        </el-card>
      </el-col>
      <el-col v-show="step==4" :span="24">
        <el-card class="box-card" style="width:80%;margin:20px auto">
          <h1>保存成功</h1>
          <el-button type="primary" @click="addAgian">继续入库</el-button>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
import CategoryCascader from '../common/material/category-cascader.vue'

import MultiUpload from '@/components/upload/multiUpload'

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {CategoryCascader, MultiUpload},
  props: {},
  data () {
    return {
      catPathSub: null,
      supplierIdSub: null,
      uploadDialogVisible: false,
      uploadImages: [],
      step: 0,
      //spu_name  spu_description  catalog_id  brand_id  weight  publish_status
      material: {},
      spu: {
        //要提交的数据
        spuName: '',
        spuDescription: '',
        wlId: 0,
        supplierId: '',
        weight: '',
        publishStatus: 0,
        decript: [], //商品详情
        images: [], //商品图集，最后sku也可以新增
        bounds: {
          //积分
          buyBounds: 0,
          growBounds: 0
        },
        baseAttrs: [], //基本属性
        skus: [] //所有sku信息
      },
      spuBaseInfoRules: {
        spuName: [
          {required: true, message: '请输入物料名字', trigger: 'blur'}
        ],
        spuDescription: [
          {required: true, message: '请编写一个简单描述', trigger: 'blur'}
        ],
        wlId: [
          {required: true, message: '请选择一个分区', trigger: 'blur'}
        ],
        supplierId: [
          {required: true, message: '请选择一个供用商', trigger: 'blur'}
        ],
        decript: [
          {required: false, message: '请上传物料详情图集', trigger: 'blur'}
        ],
        images: [
          {required: false, message: '请上传物料图片集', trigger: 'blur'}
        ],
        weight: [
          {
            type: 'number',
            required: true,
            message: '请填写正确的重量值',
            trigger: 'blur'
          }
        ]
      },
      dataResp: {
        //后台返回的所有数据
        attrGroups: [],
        baseAttrs: [],
        saleAttrs: [],
        tempSaleAttrs: [],
        tableAttrColumn: [],
        memberLevels: [],
        steped: [false, false, false, false, false]
      },
      inputVisible: [],
      inputValue: []
    }
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {
    uploadImages (val) {
      //扩展每个skus里面的imgs选项
      let imgArr = Array.from(new Set(this.spu.images.concat(val)))

      //{imgUrl:"",defaultImg:0} 由于concat每次迭代上次，有很多重复。所以我们必须得到上次+这次的总长

      this.spu.skus.forEach((item, index) => {
        let len = imgArr.length - this.spu.skus[index].images.length //还差这么多
        if (len > 0) {
          let imgs = new Array(len)
          imgs = imgs.fill({imgUrl: '', defaultImg: 0})
          this.spu.skus[index].images = item.images.concat(imgs)
        }
      })

      this.spu.images = imgArr //去重
      console.log('this.spu.skus', this.spu.skus)
    }
  },
  //方法集合
  methods: {
    addAgian () {
      location.reload()
      this.step = 0
      // this.resetSpuData();

    },
    resetSpuData () {
      this.spu = {
        spuName: '',
        spuDescription: '',
        wlId: 0,
        supplierId: '',
        weight: '',
        publishStatus: 0,
        decript: [],
        images: [],
        bounds: {
          buyBounds: 0,
          growBounds: 0
        },
        baseAttrs: [],
        skus: []
      }
    },
    /**
     handlePriceChange(scope, mpidx, e) {
     this.spu.skus[scope.$index].memberPrice[mpidx].price = e;
     },  */
    getMemberLevels () {
      this.$http({
        url: this.$http.adornUrl('/member/memberlevel/list'),
        method: 'get',
        params: this.$http.adornParams({
          page: 1,
          limit: 500
        })
      })
        .then(({data}) => {
          this.dataResp.memberLevels = data.page.list
        })
        .catch(e => {
          console.log(e)
        })
    },
    showInput (idx) {
      console.log('``````', this.view)
      this.inputVisible[idx].view = true
      // this.$refs['saveTagInput'+idx].$refs.input.focus();
    },
    checkDefaultImg (row, index, img) {
      console.log('默认图片', row, index)
      //这个图片被选中了，
      row.images[index].imgUrl = img //默认选中
      row.images[index].defaultImg = 1 //修改标志位;
      //修改其他人的标志位
      row.images.forEach((item, idx) => {
        if (idx != index) {
          row.images[idx].defaultImg = 0
        }
      })
    },
    handleInputConfirm (idx) {
      let inputValue = this.inputValue[idx].val
      if (inputValue) {
        // this.dynamicTags.push(inputValue);
        if (this.dataResp.saleAttrs[idx].valueSelect == '') {
          this.dataResp.saleAttrs[idx].valueSelect = inputValue
        } else {
          this.dataResp.saleAttrs[idx].valueSelect += ';' + inputValue
        }
      }
      this.inputVisible[idx].view = false
      this.inputValue[idx].val = ''
    },
    collectSpuBaseInfo () {
      //spuBaseForm
      this.$refs.spuBaseForm.validate(valid => {
        if (valid) {
          this.step = 1
          this.showBaseAttrs()
        } else {
          return false
        }
      })
    },
    generateSaleAttrs () {
      //把页面绑定的所有attr处理到spu里面,这一步都要做
      this.spu.baseAttrs = []
      this.dataResp.baseAttrs.forEach(item => {
        item.forEach(attr => {
          let {materialTypeId, attrValues, onematerialCount} = attr
          //跳过没有录入值的属性
          if (attrValues != '') {
            if (attrValues instanceof Array) {
              //多个值用;隔开
              attrValues = attrValues.join(';')
            }
            this.spu.baseAttrs.push({materialTypeId, attrValues, onematerialCount})
          }
        })
      })
      console.log('baseAttrs', this.spu.baseAttrs)
      this.step = 2
      this.getShowSaleAttr()
    },

    generateSkus () {
      this.step = 3
      if (!this.dataResp.steped[2]) {
        this.dataResp.baseAttrs.forEach(item => {
          item.forEach(attr => {
            this.spu.skus.push({
              materialTypeId: attr.materialTypeId,
              valueSelect: attr.attrValues,
              onematerialCount: attr.onematerialCount,
            })
          })

        })
        this.$http({
          url: this.$http.adornUrl('/material/onematerial/oneinfo'),
          method: 'post',
          data: this.$http.adornData(this.spu.skus, false)
        }).then(({data}) => {
          this.spu.skus = data.data
        })
        this.dataResp.steped[2] = true
      }
    },

    /**
     generateSkus() {
     this.step = 3;

     //根据笛卡尔积运算进行生成sku
     let selectValues = [];
     this.dataResp.tableAttrColumn = [];
     this.dataResp.tempSaleAttrs.forEach(item => {
     if (item.attrValues.length > 0) {
     selectValues.push(item.attrValues);
     this.dataResp.tableAttrColumn.push(item);
     }
     });

     let descartes = this.descartes(selectValues);
     //[["黑色","6GB","移动"],["黑色","6GB","联通"],["黑色","8GB","移动"],["黑色","8GB","联通"],
     //["白色","6GB","移动"],["白色","6GB","联通"],["白色","8GB","移动"],["白色","8GB","联通"],
     //["蓝色","6GB","移动"],["蓝色","6GB","联通"],["蓝色","8GB","移动"],["蓝色","8GB","联通"]]
     console.log("生成的组合", JSON.stringify(descartes));
     //有多少descartes就有多少sku
     let skus = [];

     descartes.forEach((descar, descaridx) => {
     let attrArray = []; //sku属性组
     descar.forEach((de, index) => {
     //构造saleAttr信息
     let saleAttrItem = {
     materialTypeId: this.dataResp.tableAttrColumn[index].materialTypeId,
     materialTypeName: this.dataResp.tableAttrColumn[index].materialTypeName,
     attrValue: de
     };
     attrArray.push(saleAttrItem);
     });
     //先初始化几个images，后面的上传还要加
     let imgs = [];
     this.spu.images.forEach((img, idx) => {
     imgs.push({ imgUrl: "", defaultImg: 0 });
     });

     //会员价，也必须在循环里面生成，否则会导致数据绑定问题
     let memberPrices = [];
     if (this.dataResp.memberLevels.length > 0) {
     for (let i = 0; i < this.dataResp.memberLevels.length; i++) {
     if (this.dataResp.memberLevels[i].priviledgeMemberPrice == 1) {
     memberPrices.push({
     id: this.dataResp.memberLevels[i].id,
     name: this.dataResp.memberLevels[i].name,
     price: 0
     });
     }
     }
     }
     //;descaridx，判断如果之前有就用之前的值;
     let res = this.hasAndReturnSku(this.spu.skus, descar);
     if (res === null) {
     skus.push({
     attr: attrArray,
     skuName: this.spu.spuName + " " + descar.join(" "),
     price: 0,
     skuTitle: this.spu.spuName + " " + descar.join(" "),
     skuSubtitle: "",
     images: imgs,
     descar: descar,
     fullonematerialCount: 0,
     disonematerialCount: 0,
     onematerialCountStatus: 0,
     fullPrice: 0.0,
     reducePrice: 0.0,
     priceStatus: 0,
     memberPrice: new Array().concat(memberPrices)
     });
     } else {
     skus.push(res);
     }
     });
     this.spu.skus = skus;
     console.log("结果!!!", this.spu.skus, this.dataResp.tableAttrColumn);
     },
     //判断如果包含之前的sku的descar组合，就返回这个sku的详细信息；
     hasAndReturnSku(skus, descar) {
     let res = null;
     if (skus.length > 0) {
     for (let i = 0; i < skus.length; i++) {
     if (skus[i].descar.join(" ") == descar.join(" ")) {
     res = skus[i];
     }
     }
     }
     return res;
     },  */
    getShowSaleAttr () {
      //获取当前分类可以使用的销售属性
      if (!this.dataResp.steped[1]) {
        this.$http({
          url: this.$http.adornUrl(
            `/material/materialtype/sale/list/${this.spu.wlId}`
          ),
          method: 'get',
          params: this.$http.adornParams({
            page: 1,
            limit: 500
          })
        }).then(({data}) => {
          this.dataResp.saleAttrs = data.page.list
          data.page.list.forEach(item => {
            this.dataResp.tempSaleAttrs.push({
              materialTypeId: item.materialTypeId,
              attrValues: [],
              materialTypeName: item.materialTypeName
            })
            this.inputVisible.push({view: false})
            this.inputValue.push({val: ''})
          })
          this.dataResp.steped[1] = true
        })
      }
    },
    showBaseAttrs () {
      if (!this.dataResp.steped[0]) {
        this.$http({
          url: this.$http.adornUrl(
            `/material/shelf/${this.spu.wlId}/withMaterialType`
          ),
          method: 'get',
          params: this.$http.adornParams({})
        }).then(({data}) => {
          //先对表单的baseAttrs进行初始化
          if (data.data != undefined) {
            data.data.forEach(item => {
              let attrArray = []
              if (item.materialTypeEntities != undefined) {
                item.materialTypeEntities.forEach(attr => {
                  attrArray.push({
                    materialTypeId: attr.materialTypeId,
                    attrValues: '',
                    onematerialCount: attr.onematerialCount
                  })
                })
              }
              this.dataResp.baseAttrs.push(attrArray)
            })
          }

          this.dataResp.steped[0] = 0
          this.dataResp.attrGroups = data.data
        })
      }
    },

    submitSkus () {
      this.$confirm('将要提交库存数据，需要一小段时间，是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.material = {
            materialName: this.spu.spuName,
            materialDescription: this.spu.spuDescription,
            wlId: this.spu.wlId,
            weight: this.spu.weight,
            materialImageInfo: this.spu.decript,
            supplierId: this.spu.supplierId,
            oneMaterial: this.spu.skus
          }
          console.log('~~~~~', JSON.stringify(this.material))
          this.$http({
            url: this.$http.adornUrl('/material/material/save'),
            method: 'post',
            data: this.$http.adornData(this.material, false)
          }).then(({data}) => {
            if (data.code == 0) {
              this.$message({
                type: 'success',
                message: '入库成功!'
              })
              this.step = 4
            } else {
              this.$message({
                type: 'error',
                message: '保存失败，原因【' + data.msg + '】'
              })
            }
          })
        })
        .catch(e => {
          console.log(e)
          this.$message({
            type: 'info',
            message: '已取消'
          })
        })
    },
    //笛卡尔积运算
    /**
     descartes(list) {

     //parent上一级索引;onematerialCount指针计数

     var point = {};

     var result = [];
     var pIndex = null;
     var temponematerialCount = 0;
     var temp = [];

     //根据参数列生成指针对象
     for (var index in list) {
     if (typeof list[index] == "object") {
     point[index] = { parent: pIndex, onematerialCount: 0 };
     pIndex = index;
     }
     }

     //单维度数据结构直接返回
     if (pIndex == null) {
     return list;
     }

     //动态生成笛卡尔积
     while (true) {
     for (var index in list) {
     temponematerialCount = point[index]["onematerialCount"];
     temp.push(list[index][temponematerialCount]);
     }

     //压入结果数组
     result.push(temp);
     temp = [];

     //检查指针最大值问题
     while (true) {
     if (point[index]["onematerialCount"] + 1 >= list[index].length) {
     point[index]["onematerialCount"] = 0;
     pIndex = point[index]["parent"];
     if (pIndex == null) {
     return result;
     }

     //赋值parent进行再次检查
     index = pIndex;
     } else {
     point[index]["onematerialCount"]++;
     break;
     }
     }
     }
     } */
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created () {
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted () {
    this.catPathSub = PubSub.subscribe('catPath', (msg, val) => {
      this.spu.wlId = val[val.length - 1]
    })
    this.supplierIdSub = PubSub.subscribe('supplierId', (msg, val) => {
      this.spu.supplierId = val
    })
    this.getMemberLevels()
  },
  beforeCreate () {
  }, //生命周期 - 创建之前
  beforeMount () {
  }, //生命周期 - 挂载之前
  beforeUpdate () {
  }, //生命周期 - 更新之前
  updated () {
  }, //生命周期 - 更新之后
  beforeDestroy () {
    PubSub.unsubscribe(this.catPathSub)
    PubSub.unsubscribe(this.supplierIdSub)
  }, //生命周期 - 销毁之前
  destroyed () {
  }, //生命周期 - 销毁完成
  activated () {
  } //如果页面有keep-alive缓存功能，这个函数会触发
}
</script>
<style scoped>
</style>
