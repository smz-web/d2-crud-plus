<template>
  <span>
    <template v-if="type === 'text'">
      <span v-for="(item) in items" :key="item[dict.value]">{{item[dict.label]}}</span>
    </template>
    <template v-else >
      <el-tag class='tag-item  d2-mr-5 d2-mb-2 d2-mt-2' v-for="(item) in items" :key="item[dict.value]"  size="small"  :type="item[dict.color]" >
        {{item[dict.label]}}
      </el-tag>
    </template>
  </span>
</template>

<script>
import dict from '../../utils/util.dicts'
// value格式化展示组件
export default {
  name: 'values-format',
  props: {
    // 值
    value: {
      require: true
    },
    // 是否多选
    multiple: { default: true, require: false },
    // value的分隔符<br/>
    // 多选时，如果value为string，则以该分隔符分割成多个展示<br/>
    // 传入空字符串，表示不分割<br/>
    separator: { default: ',', require: false },
    // 数据字典<br/>
    // {url:'xxx',data:[],value:'',label:'',children:''}
    dict: {
      type: Object,
      require: false,
      default () {
        return {}
      }
    },
    // 颜色，【primary, success, warning, danger ,info】
    color: {
      require: false,
      default: 'primary'
    },
    // 展示类型【text, tag】
    type: {
      default: 'tag' // 可选【text,tag】
    }
  },
  data () {
    return {
      dictCopy: {},
      dictDataMap: {}
    }
  },
  computed: {
    items () {
      if (this.value == null || this.value === '') {
        return []
      }
      let dictDataMap = this.dictDataMap
      let valueArr = []
      let options = []
      if (typeof (this.value) === 'string' && this.multiple && this.separator != null && this.separator !== '') {
        valueArr = this.value.split(this.separator)
      } else if (this.value instanceof Array) {
        // 本来就是数组的
        valueArr = this.value
      } else {
        valueArr = [this.value]
      }

      let dict = this.dict
      // 没有字典，直接显示值
      if (dictDataMap == null || Object.keys(dictDataMap).length === 0) {
        for (let str of valueArr) {
          let item = {}
          item[dict.value] = str
          item[dict.label] = str
          item[dict.color] = this.color
          options.push(item)
        }
        return options
      }
      // 根据字典展示
      for (let str of valueArr) {
        let item = this.dictDataMap[str]
        if (item != null) {
          options.push(item)
        } else {
          item = {}
          item[dict.value] = str
          item[dict.label] = str
          item[dict.color] = this.color
          options.push(item)
        }
      }
      return options
    }
  },
  created () {
    if (this.dict == null) {
      this.dict = {}
    }
    dict.mergeDefault(this.dict)
    dict.get(this.dict).then((data) => {
      let dataMap = this.dict.dataMap
      if (dataMap == null && data != null && data.length > 0) {
        dataMap = {}
        this.putAll(dataMap, data, this.dict.isTree)
        // this.$set(this, 'dictData', data)
        // dict.putCache(this.dict.mapCacheName, dataMap)
        this.dict.dataMap = dataMap
        // console.log('dictDataMap', this.dict, dataMap)
      }
      this.$set(this, 'dictDataMap', dataMap)
    })
  },
  methods: {
    putAll (map, list, isTree) {
      let valueName = this.dict.value
      let childrenName = this.dict.children
      for (let item of list) {
        map[item[valueName]] = item
        // console.log('isTree', isTree, item[childrenName], childrenName, item, this.dict)
        if (isTree && item[childrenName] != null) {
          this.putAll(map, item[childrenName], isTree)
        }
      }
    }
  }
}
</script>
<style >
  .d2-mb-2{margin-bottom: 2px}
  .d2-mt-2{margin-top: 2px;}
</style>
