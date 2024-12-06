<template>
  <div class="a1">
    <el-row :gutter="20">
      <el-col :span="4">
        <!--   <el-switch v-model="draggable" active-text="开启拖拽" inactive-text="关闭拖拽"></el-switch> -->
        <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
        <el-button type="danger" @click="batchDelete">批量删除</el-button>
        <el-tree
          ref="menuTree"
          :allow-drop="allowDrop"
          :data="menus"
          :default-expanded-keys="expandedKey"
          :draggable="draggable"
          :expand-on-click-node="false"
          :props="defaultProps"
          node-key="wlId"
          show-checkbox
          @node-drop="handleDrop"
        >
      <span slot-scope="{ node, data }" class="custom-tree-node">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <=2"
            size="mini"
            type="text"
            @click="() => append(data)"
          >添加</el-button>
          <el-button size="mini" type="text" @click="edit(data)">修改</el-button>
          <el-button
            v-if="node.childNodes.length==0"
            size="mini"
            type="text"
            @click="() => remove(node, data)"
          >删除</el-button>
        </span>
      </span>
        </el-tree>

        <el-dialog
          :close-on-click-modal="false"
          :title="title"
          :visible.sync="dialogVisible"
          width="30%"
        >
          <el-form :model="category">
            <el-form-item label="分区名称">
              <el-input v-model="category.name" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="图标">
              <el-input v-model="category.icon" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="计量单位">
              <el-input v-model="category.goodsUnit" autocomplete="off"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
        </el-dialog>
      </el-col>

      <el-col :span="20">
        <div id="chart1" class="chart-box"></div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
import * as echarts from 'echarts'

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {echarts},
  props: {},
  data () {
    return {
      chart1data: [],
      chart1: null,
      pCid: [],
      draggable: false,
      updateNodes: [],
      maxLevel: 0,
      title: '',
      dialogType: '', //edit,add
      category: {
        name: '',
        parentWlid: 0,
        wlLevel: 0,
        showStatus: 1,
        sort: 0,
        goodsUnit: '',
        icon: '',
        wlId: null
      },
      dialogVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },

  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {

    // 图
    initchart1 () {
      var option = {}
      /**     var option = {
       series: [
       {
       type: 'treemap',
       data: this.chart1data
       }
       ]
       };*/
      this.chart1 = echarts.init(document.getElementById('chart1'))
      this.chart1.showLoading()
      this.chart1.hideLoading()

      function getLevelOption () {
        return [
          {
            itemStyle: {
              borderColor: '#777',
              borderWidth: 0,
              gapWidth: 1
            },
            upperLabel: {
              show: false
            }
          },
          {
            itemStyle: {
              borderColor: '#555',
              borderWidth: 5,
              gapWidth: 1
            },
            emphasis: {
              itemStyle: {
                borderColor: '#ddd'
              }
            }
          },
          {
            colorSaturation: [0.35, 0.5],
            itemStyle: {
              borderWidth: 5,
              gapWidth: 1,
              borderColorSaturation: 0.6
            }
          }
        ]
      }
      this.chart1.setOption(option = {
        title: {
          text: '仓库一览图',
          left: 'center'
        },
        tooltip: {
          formatter: function (info) {
            var value = info.value
            var treePathInfo = info.treePathInfo
            var treePath = []
            for (var i = 1; i < treePathInfo.length; i++) {
              treePath.push(treePathInfo[i].name)
            }
            return [
              '<div class="tooltip-title">' +
              echarts.format.encodeHTML(treePath.join('/')) +
              '</div>',
              '货架数量: ' + echarts.format.addCommas(value) + '个'
            ].join('')
          }
        },
        series: [
          {
            name: '仓库层级关系',
            type: 'treemap',
            visibleMin: 300,
            label: {
              show: true,
              formatter: '{b}'
            },
            upperLabel: {
              show: true,
              height: 30
            },
            itemStyle: {
              borderColor: '#fff'
            },
            levels: getLevelOption(),
            data: this.chart1data
          }
        ]
      })
      window.addEventListener('resize', () => {
        this.chart1.resize()
      })
    },
    getchart1data () {
      this.$http({
        url: this.$http.adornUrl('/material/warelocation/list/echarts'),
        method: 'get'
      }).then(({data}) => {
        this.chart1data = data.data
        this.initchart1()
      })
    },

    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/material/warelocation/list/tree'),
        method: 'get'
      }).then(({data}) => {
        console.log('成功获取到菜单数据...', data.data)
        this.menus = data.data
      })
    },
    batchDelete () {
      let wlIds = []
      let checkedNodes = this.$refs.menuTree.getCheckedNodes()
      console.log('被选中的元素', checkedNodes)
      for (let i = 0; i < checkedNodes.length; i++) {
        wlIds.push(checkedNodes[i].wlId)
      }
      this.$confirm(`是否批量删除【${wlIds}】菜单?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl('/material/warelocation/delete'),
            method: 'post',
            data: this.$http.adornData(wlIds, false)
          }).then(({data}) => {
            this.$message({
              message: '菜单批量删除成功',
              type: 'success'
            })
            this.getMenus()
          })
        })
        .catch(() => {
        })
    },
    batchSave () {
      this.$http({
        url: this.$http.adornUrl('/material/warelocation/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单顺序等修改成功',
          type: 'success'
        })
        //刷新出新的菜单
        this.getMenus()
        //设置需要默认展开的菜单
        this.expandedKey = this.pCid
        this.updateNodes = []
        this.maxLevel = 0
        // this.pCid = 0;
      })
    },
    handleDrop (draggingNode, dropNode, dropType, ev) {
      console.log('handleDrop: ', draggingNode, dropNode, dropType)
      //1、当前节点最新的父节点id
      let pCid = 0
      let siblings = null
      if (dropType == 'before' || dropType == 'after') {
        pCid =
          dropNode.parent.data.wlId == undefined
            ? 0
            : dropNode.parent.data.wlId
        siblings = dropNode.parent.childNodes
      } else {
        pCid = dropNode.data.wlId
        siblings = dropNode.childNodes
      }
      this.pCid.push(pCid)

      //2、当前拖拽节点的最新顺序，
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.wlId == draggingNode.data.wlId) {
          //如果遍历的是当前正在拖拽的节点
          let wlLevel = draggingNode.level
          if (siblings[i].level != draggingNode.level) {
            //当前节点的层级发生变化
            wlLevel = siblings[i].level
            //修改他子节点的层级
            this.updateChildNodeLevel(siblings[i])
          }
          this.updateNodes.push({
            wlId: siblings[i].data.wlId,
            sort: i,
            parentWlid: pCid,
            wlLevel: wlLevel
          })
        } else {
          this.updateNodes.push({wlId: siblings[i].data.wlId, sort: i})
        }
      }

      //3、当前拖拽节点的最新层级
      console.log('updateNodes', this.updateNodes)
    },
    updateChildNodeLevel (node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data
          this.updateNodes.push({
            wlId: cNode.wlId,
            wlLevel: node.childNodes[i].level
          })
          this.updateChildNodeLevel(node.childNodes[i])
        }
      }
    },
    allowDrop (draggingNode, dropNode, type) {
      //1、被拖动的当前节点以及所在的父节点总层数不能大于3

      //1）、被拖动的当前节点总层数
      console.log('allowDrop:', draggingNode, dropNode, type)
      //
      this.countNodeLevel(draggingNode)
      //当前正在拖动的节点+父节点所在的深度不大于3即可
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1
      console.log('深度：', deep)

      //   this.maxLevel
      if (type == 'inner') {
        // console.log(
        //   `this.maxLevel：${this.maxLevel}；draggingNode.data.wlLevel：${draggingNode.data.wlLevel}；dropNode.level：${dropNode.level}`
        // );
        return deep + dropNode.level <= 3
      } else {
        return deep + dropNode.parent.level <= 3
      }
    },
    countNodeLevel (node) {
      //找到所有子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level
          }
          this.countNodeLevel(node.childNodes[i])
        }
      }
    },
    edit (data) {
      console.log('要修改的数据', data)
      this.dialogType = 'edit'
      this.title = '修改分区'
      this.dialogVisible = true

      //发送请求获取当前节点最新的数据
      this.$http({
        url: this.$http.adornUrl(`/material/warelocation/info/${data.wlId}`),
        method: 'get'
      }).then(({data}) => {
        //请求成功
        console.log('要回显的数据', data)
        this.category.name = data.data.name
        this.category.wlId = data.data.wlId
        this.category.icon = data.data.icon
        this.category.goodsUnit = data.data.goodsUnit
        this.category.parentWlid = data.data.parentWlid
        this.category.wlLevel = data.data.wlLevel
        this.category.sort = data.data.sort
        this.category.showStatus = data.data.showStatus
        /**
         *         parentWlid: 0,
         wlLevel: 0,
         showStatus: 1,
         sort: 0,
         */
      })
    },
    append (data) {
      console.log('append', data)
      this.dialogType = 'add'
      this.title = '添加分区'
      this.dialogVisible = true
      this.category.parentWlid = data.wlId
      this.category.wlLevel = data.wlLevel * 1 + 1
      this.category.wlId = null
      this.category.name = ''
      this.category.icon = ''
      this.category.goodsUnit = ''
      this.category.sort = 0
      this.category.showStatus = 1
    },

    submitData () {
      if (this.dialogType == 'add') {
        this.addCategory()
      }
      if (this.dialogType == 'edit') {
        this.editCategory()
      }
    },
    //修改三级分类数据
    editCategory () {
      var {wlId, name, icon, goodsUnit} = this.category
      this.$http({
        url: this.$http.adornUrl('/material/warelocation/update'),
        method: 'post',
        data: this.$http.adornData({wlId, name, icon, goodsUnit}, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单修改成功',
          type: 'success'
        })
        //关闭对话框
        this.dialogVisible = false
        //刷新出新的菜单
        this.getMenus()
        //设置需要默认展开的菜单
        this.expandedKey = [this.category.parentWlid]
      })
    },
    //添加三级分类
    addCategory () {
      console.log('提交的三级分类数据', this.category)
      this.$http({
        url: this.$http.adornUrl('/material/warelocation/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单保存成功',
          type: 'success'
        })
        //关闭对话框
        this.dialogVisible = false
        //刷新出新的菜单
        this.getMenus()
        //设置需要默认展开的菜单
        this.expandedKey = [this.category.parentWlid]
      })
    },

    remove (node, data) {
      var ids = [data.wlId]
      this.$confirm(`是否删除【${data.name}】菜单?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl('/material/warelocation/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            this.$message({
              message: '菜单删除成功',
              type: 'success'
            })
            //刷新出新的菜单
            this.getMenus()
            //设置需要默认展开的菜单
            this.expandedKey = [node.parent.data.wlId]
          })
        })
        .catch(() => {
        })

      console.log('remove', node, data)
    }
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created () {
    this.getMenus()
    this.getchart1data()
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted () {
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
  }, //生命周期 - 销毁之前
  destroyed () {
  }, //生命周期 - 销毁完成
  activated () {
    // 由于给echart添加了resize事件, 在组件激活时需要重新resize绘画一次, 否则出现空白bug
    if (this.chart1) {
      this.chart1.resize()
    }
  } //如果页面有keep-alive缓存功能，这个函数会触发
}
</script>
<style lang="scss">
.a1 {

  > .el-row {
    margin: 0 0 0 0;
    margin-top: -10px;
    margin-bottom: -10px;

    .el-col {
      padding-top: 10px;
      padding-bottom: 10px;
    }

    .el-tree {

      min-height: 250px;
      color: rgb(105, 116, 219);
      font-size: 20px;
      font-weight: 300;
    }

    .chart-box {
      min-height: 800px;

    }
  }
}


</style>
