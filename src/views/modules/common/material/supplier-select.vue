<template>
  <div>
    <el-select v-model="supplierId" clearable filterable placeholder="请选择">
      <el-option
        v-for="item in brands"
        :key="item.supplierId"
        :label="item.name"
        :value="item.supplierId"
      ></el-option>
    </el-select>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data () {
    //这里存放数据
    return {
      wlId: 0,
      brands: [
        {
          label: 'a',
          value: 1
        }
      ],
      supplierId: '',
      subscribe: null
    }
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {
    supplierId (val) {
      this.PubSub.publish('supplierId', val)
    }
  },
  //方法集合
  methods: {
    getCatBrands () {
      this.$http({
        url: this.$http.adornUrl('/material/warelocationsupplierrelation/supplier/list'),
        method: 'get',
        params: this.$http.adornParams({
          wlId: this.wlId
        })
      }).then(({data}) => {
        this.brands = data.data
      })
    }
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created () {
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted () {
    //监听三级分类消息的变化
    this.subscribe = PubSub.subscribe('catPath', (msg, val) => {
      this.wlId = val[val.length - 1]
      this.getCatBrands()
    })
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
    PubSub.unsubscribe(this.subscribe) //销毁订阅
  }, //生命周期 - 销毁之前
  destroyed () {
  }, //生命周期 - 销毁完成
  activated () {
  } //如果页面有keep-alive缓存功能，这个函数会触发
}
</script>
<style scoped>
</style>
