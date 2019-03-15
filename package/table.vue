<template>
  <div>
    <Row :gutter="16" v-if="searchFields.length">
      <Col v-bind="searchFit" v-for="(slot,index) in $slots" :key="`custom-search-${index}`">
        <slot v-if="index.indexOf('custom-search') !== -1" :name="index"/>
      </Col>
      <Col
        v-bind="searchFit"
        v-for="field in searchFields"
        :key="field.key"
        style="margin-bottom: 5px;"
      >
        <Input
          v-if="field.type === 'Input'"
          v-model="search[field.key]"
          v-bind="field.props"
          :key="field.key"
        />
        <Select
          v-else-if="field.type === 'Select'"
          v-model="search[field.key]"
          v-bind="field.props"
          :key="field.key"
        >
          <Option
            v-for="option in field.options"
            :value="option.value"
            :key="option.value"
          >{{ option.label }}</Option>
        </Select>
        <DatePicker
          v-else-if="field.type === 'Date'"
          v-model="search[field.key]"
          :type="field.dateType || 'date'"
          v-bind="field.props"
          :key="field.key"
          style="display:block;"
        />
        <Cascader
          v-else-if="field.type === 'Cascader'"
          v-model="search[field.key]"
          v-bind="field.props"
          :data="field.options"
        />
      </Col>
      <Col v-bind="searchFit" style="margin-bottom: 5px;">
        <Button type="primary" @click="searchHandle" style="margin-right:10px;">搜索</Button>
        <Button type="primary" @click="initializeParams">重置</Button>
      </Col>
    </Row>
    <Row>
      <Col span="24" class="custom-row">
        <slot></slot>
      </Col>
    </Row>
    <Table
      ref="table"
      :columns="columns"
      :data="rows"
      v-bind="tableProps"
      @on-sort-change="(column,key,order) => emitEventHandler('on-sort-change',column,key,order)"
      @on-selection-change="selection => emitEventHandler('on-selection-change',selection)"
      @on-current-change="(currentRow,oldCurrentRow) => (tableConfig['highlight-row'] || tableConfig['highlightRow']) && emitEventHandler('on-current-change',currentRow,oldCurrentRow)"
    />
    <Page
      v-if="pagination"
      style="padding: 5px 10px;text-align: right;"
      v-bind="pageProps"
      :current="pageNumber"
      :total="total"
      @on-change="pageNumber => page = emitDataHandle({ pageNumber })"
      @on-page-size-change="pageSize => page = { pageNumber: 1, pageSize }"
    />
  </div>
</template>


<script>
import {
  Row,
  Col,
  Page,
  Button,
  Table,
  Select,
  Input,
  DatePicker,
  Option,
  Cascader
} from 'iview'
//
export default {
  name: 'ZhTable',
  components: {
    Row,
    Col,
    Page,
    Button,
    Table,
    Input,
    DatePicker,
    Select,
    Option,
    Cascader
  },
  data() {
    return {
      fetchParams: {},
      page: {
        pageNumber: 1,
        pageSize: 10
      },
      search: {},
      searchFields: [],
      loadDone: false
    }
  },
  props: {
    columns: {
      type: [Array, Object],
      required: true
    },
    data: {
      type: [Array, Object],
      required: true
    },
    rowsField: {
      type: String,
      default: 'content'
    },
    totalField: {
      type: String,
      default: 'total'
    },
    pageNumberField: {
      type: String,
      default: 'pageNumber'
    },
    // 响应式
    searchFit: {
      type: Object,
      default: () => {
        return {
          xs: 24,
          sm: 12,
          md: 6,
          lg: 4
        }
      }
    },
    // 分页
    pagination: {
      type: [Boolean, Object],
      default: false
    },
    // 表格配置项
    tableConfig: {
      type: Object
    },
    extendSearchFields: {
      type: Array,
      default: () => []
    }
  },
  watch: {
    // 监听 分页变化
    page() {
      this.$emit('on-change', this.emitDataHandle())
    }
  },
  computed: {
    rows() {
      return this.data[this.rowsField]
    },
    total() {
      return this.data[this.totalField]
    },
    pageNumber() {
      return this.data[this.pageNumberField] || this.page.pageNumber || 1
    },
    // 表格配置
    tableProps() {
      let defatult = {
        stripe: true
      }
      return this.tableConfig &&
        this.isObject(this.tableConfig) &&
        Object.keys(this.tableConfig).length
        ? { ...defatult, ...this.tableConfig }
        : defatult
    },
    // 分页配置
    pageProps() {
      let defatult = {
        size: 'small',
        showTotal: true,
        showElevator: true,
        showSizer: true
      }
      return this.pagination &&
        this.isObject(this.pagination) &&
        Object.keys(this.pagination).length
        ? { ...defatult, ...this.pagination }
        : defatult
    }
  },
  methods: {
    // 事件触发器
    emitEventHandler(event, ...args) {
      this.$emit(event, ...args)
    },
    emitDataHandle(params = {}) {
      // Object.assign({}, this.page, this.search, params)
      return this.Object2NotNull({ ...this.page, ...this.search, ...params })
    },
    // 刷新表格
    refresh() {
      this.emitEventHandler('on-refresh', this.emitDataHandle())
    },
    searchHandle() {
      this.page.pageNumber = 1
      this.emitEventHandler('on-search', this.emitDataHandle())
    },
    initializeParams() {
      this.page.pageNumber = 1
      Object.keys(this.search).forEach(k => (this.search[k] = null))
      // Object.keys(this.search).map(k => this.search[k] = null)
      if (this.page.pageNumber === 1) {
        this.$emit('on-change', this.emitDataHandle())
      }
    },
    isObject(value) {
      const type = typeof value
      return value != null && (type === 'object' || type === 'function')
    },

    /**
     * @param {*} obj 对象
     * @description *
     * 处理对象参数值，排除对象参数值为”“、null、undefined，并返回一个新对象
     */
    Object2NotNull(obj) {
      let param = {}
      if (obj === null || obj === undefined || obj === '') return param
      for (var key in obj) {
        if (
          Object.prototype.toString.call(obj[key]).slice(8, -1) === 'Object'
        ) {
          param[key] = this.Object2NotNull(obj[key])
        } else if (
          obj[key] !== null &&
          obj[key] !== undefined &&
          obj[key] !== ''
        ) {
          param[key] = obj[key]
        }
      }
      return param
    },
    getSearchFields() {
      // return new Promise((resolve, reject) => {})
      let temp = []
      // 合并额外字段
      let searchFields = [...this.columns, ...this.extendSearchFields]
      searchFields.forEach(async (item, index) => {
        let { search } = item
        if (search && search.hasOwnProperty('type')) {
          let o = {
            title: item.title,
            key: item.key,
            type: search.type,
            sort: search.sort || index
          }
          if (search.hasOwnProperty('props')) {
            o.props = search.props
          }
          if (search.type === 'Select' || search.type === 'Cascader') {
            let _options = null
            if (Array.isArray(search.options)) {
              _options = search.options
            } else if (typeof search.options === 'function') {
              const res = await search.options(...(search.params || []))
              if (search.format && typeof search.format === 'function') {
                _options = search.format.call(this, res)
              } else {
                _options = res
              }
            }
            o.options = _options
          }
          if (search.type === 'Date') {
            o.dateType = search.dateType
          }
          this.$set(
            this.search,
            item.key,
            search.type === 'Cascader' ? [] : null
          )
          temp.push(o)
        }
      })
      this.searchFields = temp.sort((a, b) => a.sort - b.sort)
    },
    initialize() {
      // 获取需要查询的字段
      this.getSearchFields()
      // 初始化查询字段
      // this.initializeParams()
    }
  },
  created() {
    this.initialize()
  },
  mounted() {
    this.$emit('on-table-mounted', this.$refs.table)
  }
}
</script>

<style lang="less" scoped>
</style>
