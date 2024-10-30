<template>
    <div class="mod-demo-echarts">
    <el-row :gutter="20">
      
      
      <el-col :span="8">
        <el-card class="a1">
          <el-col class="a2" span="17">
          <h2 class="card_h2">今日新增物料申请</h2>
          
          <el-tag class="b2">{{this.dataShow.todaymbCount}}</el-tag>
          </el-col>
          <el-col span="7">
            <br>
            <br>
            <el-button type="primary" icon="el-icon-edit" circle></el-button>
            </el-col>
        </el-card>
      </el-col>
       <el-col :span="8">
         <el-card class="a1">
           <el-col class="a2" span="17">
          <h2 class="card_h2">今日新增入库单</h2>
            <el-tag class="b2">{{this.dataShow.todayibCount}}</el-tag>
          </el-col>
            <el-col span="7">
            <br>
            <br>
             <el-button type="warning" icon="el-icon-star-off" circle></el-button>
            </el-col>
        </el-card>
      </el-col>
      <el-col :span="8">
        <el-card class="a1">
          <el-col class="a2" span="17">
          <h2 class="card_h2">今日新增出库单</h2>
          <el-tag class="b2">{{this.dataShow.todayobCount}}</el-tag>
          </el-col>
            <el-col span="7">
            <br>
            <br>
            <el-button type="primary" icon="el-icon-edit" circle></el-button>
            </el-col>
        </el-card>   
      </el-col>
      
        <el-col :span="8">
        <el-card class="a1">
           <el-col class="a2" span="15">
          <h2 class="card_h2">待处理物料申请</h2>
         
          <el-tag class="b2">{{this.dataShow.todombCount}}</el-tag>
          </el-col>
               <el-col span="9"> 
          <el-progress type="circle" :percentage=" Math.round((this.dataShow.todaymbCount/this.dataShow.todombCount)*100)"></el-progress>
          </el-col>
        </el-card>
      </el-col>
      <el-col :span="8">
        <el-card class="a1">
          <el-col class="a2" span="15">
          <h2 class="card_h2">待上架入库单</h2>
            <el-tag class="b2">{{this.dataShow.todoibCount}}</el-tag>
          </el-col>
           <el-col span="9"> 
          <el-progress type="circle" :percentage="Math.round((this.dataShow.todayibCount/this.dataShow.todoibCount)*100)"></el-progress>
          </el-col>
        </el-card>
      </el-col>
      <el-col :span="8">
         <el-card class="a1">
           <el-col class="a2" span="15">
          <h2 class="card_h2">待处理出库单</h2>
          
         <el-tag class="b2">{{this.dataShow.todayobCount}}</el-tag>
          </el-col>
          <el-col span="9"> 
          <el-progress type="circle" :percentage="Math.round((this.dataShow.todayobCount/this.dataShow.todayobCount)*100)"></el-progress>
          </el-col>
        </el-card>
      </el-col>
       <el-col :span="16">
        <el-card>
          <div id="chartDom" class="chart-box"></div>
        </el-card>
      </el-col>
      <el-col :span="8">
        <el-card>
          <div id="chartDom2" class="chart-box"></div>
        </el-card>
      </el-col>
    </el-row>
    
      </div>
</template>

<script>
import * as echarts from 'echarts';

  export default {
     data () {
      return {
        days: [],
        count: [],
        dataShow: {
          todaymbCount: 0,
          todombCount: 0,
          todayibCount: 0,
          todoibCount: 0,
          todayobCount: 0,
          todoobCount: 0,
        },
        chartDom2data:[{value:0,name:""}],
        chartDom: null,
        chartDom2: null,
      }
    },
      mounted () {
       this.getchartDomdata(); 
       this.getchartDom2data();
      
 
    },
    activated () {
      // 由于给echart添加了resize事件, 在组件激活时需要重新resize绘画一次, 否则出现空白bug
      if (this.chartDom) {
        this.chartDom.resize()
      }
      if (this.chartDom2) {
        this.chartDom2.resize()
      }
    },
    methods: {
      getchartDomdata(){
        this.$http({
          url: this.$http.adornUrl("/material/mbill/mouthmbill"),
          method: "get",
        }).then(({ data }) => {
          data.data.forEach(element => {
            this.days.push(element.days);
            this.count.push(element.count)
          });
          this.initchartDom();
        }); 
      },
      getchartDom2data(){
        this.$http({
          url: this.$http.adornUrl("/material/mbill/mouthmbillbingtu"),
          method: "get",
        }).then(({ data }) => {
          data.data.forEach(element => {
            this.chartDom2data.push({value:element.value,name:element.name});
          });
          this.initchartDom2();
        }); 
      },
      // 折线图
      initchartDom () {
        var option = {
            title: {
            text: '近一个月物料申请单数量的趋势'
            },
            xAxis: {
            type: 'category',
            data: this.days
            },
            yAxis: {
            type: 'value'
            },
            series: [
            {
              data: this.count,
              type: 'line',
              smooth: true
            }
            ]
        };
        this.chartDom = echarts.init(document.getElementById('chartDom'))
        this.chartDom.setOption(option)
        window.addEventListener('resize', () => {
          this.chartDom.resize()
        })
      },
      //饼图
      initchartDom2 () {
        var option = {
            legend: {
              top: 'bottom'
            },
            toolbox: {
              show: true,
              feature: {
                mark: { show: true },
                dataView: { show: true, readOnly: false },
                restore: { show: true },
                saveAsImage: { show: true }
              }
            },
            series: [
              {
                name: '近一月分区物料申请数量详细信息',
                type: 'pie',
                radius: [25, 125],
                center: ['50%', '50%'],
                roseType: 'area',
                itemStyle: {
                  borderRadius: 8
                },
                data: this.chartDom2data
              }
            ]
        };
        this.chartDom = echarts.init(document.getElementById('chartDom2'))
        this.chartDom.setOption(option)
        window.addEventListener('resize', () => {
          this.chartDom.resize()
        })
      },
      gettodaymbCount(){
        this.$http({
        url: this.$http.adornUrl("/material/mbill/todaymbill"),
        method: "get",
      }).then(({ data }) => {
        this.dataShow.todaymbCount = data.taday;
        this.dataShow.todombCount = data.todo;
      });
      },
     gettodayibCount(){
        this.$http({
        url: this.$http.adornUrl("/stock/stock/todaybill"),
        method: "get",
      }).then(({ data }) => {
        this.dataShow.todayibCount = data.taday;
        this.dataShow.todoibCount = data.todo;
      });
      },
      gettodayobCount(){
        this.$http({
        url: this.$http.adornUrl("/stock/outbill/todayoutbill"),
        method: "get",
      }).then(({ data }) => {
        this.dataShow.todayobCount = data.taday;
        this.dataShow.todoobCount = data.todo;
      });
      },
      },
      created() {
    this.gettodaymbCount();
     this.gettodayibCount();
       this.gettodayobCount();
  },
  }
</script>

<style lang="scss">
  
  .mod-demo-echarts {
    > .el-alert {
      margin-bottom: 10px;
    }
    > .el-row {
      margin-top: -10px;
      margin-bottom: -10px;
      .el-col {
        padding-top: 10px;
        padding-bottom: 10px;
      }
      .card_h2{
        color: rgb(29, 226, 160);
      }    
    }
    .chart-box {
      min-height: 400px;
    }
   .b2{
        color: rgb(236, 45, 71);
        font-size: 30px;
        font-weight: 500;
        min-width: 121px;
        padding-left: 50px;

    }
    .a1{
      padding-left: 60px;
    }

  }
</style>

