<template>
  <div id="app">
    <ZhTable
      :pagination="pagination"
      :columns="columns"
      :data="data"
      :extendSearchFields="extendSearchFields"
      :tableConfig="{highlightRow:true}"
      @on-change="changeHandle"
      @on-table-mounted="tableMounted"
      @on-search="searchHandle"
      @on-sort-change="sortChangeHandle"
      @on-selection-change="selectChange"
      @on-current-change="currentChange"
    >
      <Input slot="custom-search1"/>
      <Input slot="custom-search2"/>
      <Input slot="custom-sear2ch2"/>
    </ZhTable>
  </div>
</template>

<script>
import { Input } from 'iview'
export default {
  components: { Input },
  data() {
    return {
      // api 遵循 iview page 的属性 并转为驼峰命名
      pagination: {},
      columns: [
        {
          title: 'Id',
          key: 'id',
          search: {
            type: 'Input',
            sort: 12,
            props: {
              placeholder: '请输入ID'
            }
          }
        },
        {
          title: 'Name',
          key: 'name'
        },
        {
          title: 'Gender',
          key: 'gender',
          search: {
            type: 'Select',
            options: [
              { value: '', label: '请选择11111' },
              { value: 'male', label: '男' },
              { value: 'female', label: '女' }
            ]
            // options: UserApi.getOptions,
            // formart(res) {
            //   return [...res, { value: '123', label: '456' }]
            // }
          }
        }
      ],
      /**
       * data 需要包含 total 分页信息.
       * 默认字段: rows total
       * 可自定义 数据字段
       */
      data: {},
      table: null,
      extendSearchFields: [
        {
          key: 'keyword',
          search: {
            type: 'Cascader',
            options: [
              {
                value: 'beijing',
                label: '北京',
                children: [
                  {
                    value: 'gugong',
                    label: '故宫'
                  },
                  {
                    value: 'tiantan',
                    label: '天坛'
                  },
                  {
                    value: 'wangfujing',
                    label: '王府井'
                  }
                ]
              }
            ]
          }
        }
      ]
    }
  },
  methods: {
    /**
     * @description 处理搜索
     * @param 分页信息及查询参数集合
     */
    searchHandle(fetchParams) {
      console.log(fetchParams, 'searchHandle')
      this.getTableData(fetchParams)
    },
    // 处理多选单选
    selectChange(selection) {
      console.log(selection)
    },
    currentChange(currentRow, oldCurrentRow) {
      console.log(currentRow, oldCurrentRow)
    },
    // 处理排序
    sortChangeHandle({ column, key, order }) {
      console.log(column, key, order)
    },
    // 处理分页
    changeHandle(fetchParams) {
      this.getTableData(fetchParams)
    },
    // table mounted 回调,params: table => 组件实例
    tableMounted(table) {
      this.table = table
    },
    getTableData(params = { pageNumber: 1, pageSize: 10 }) {
      console.log(params, 'getTableData')
      
    },
  },
  mounted() {
    console.log('mountedmounted')
    this.getTableData()
  }
}
</script>