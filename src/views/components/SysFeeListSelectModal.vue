<!--
 * @Author: 赵峰
 * @Date: 2021-09-13 14:19:16
 * @LastEditTime: 2021-09-13 16:24:43
 * @LastEditors: 赵峰
 * @Descripttion: 系统内试验项目列表弹窗组件
 * @FilePath: \hifar-platform-client\src\views\components\SysFeeListSelectModal.vue
-->
<template>
  <h-modal
    destroyOnClose
    inner
    fullScreen
    :title="title"
    :visible="visible"
    :width="800"
    :confirmLoading="confirmLoading"
    :getContainer="getContainer"
    @cancel="handleCancel"
    @submit="handleOk"
  >
    <h-card fixed :bordered="false">
      <h-search
        v-model="queryParams"
        slot="search-form"
        size="small"
        :showToggleButton="false"
        :data="searchBar"
        @change="refresh(true)"
      />
      <h-vex-table
        ref='feeTable'
        slot="content"
        :columns="columns"
        :data="loadData"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, selectedRows: selectedRows, onChange: onSelected }"
      ></h-vex-table>
    </h-card>
  </h-modal>
</template>

<script>
import { postAction } from '@/api/manage'
import { isArray } from 'lodash'
export default {
  inject: {
    getContainer: {
      default: () => {
        return () => document.body
      },
    },
  },
  data() {
    return {
      title: '试验项目列表',
      visible: false,
      confirmLoading: false,
      queryParams: {},
      url: {
        list: '/HfResCustPriceBusiness/listPageProject',
      },
      searchBar: [
        {
          title: '项目名称',
          formType: 'input',
          key: 'c_unitName_7',
        },
      ],
      columns: [
        {
          title: '试验项目编码',
          align: 'center',
          dataIndex: 'unitCode',
        },
        {
          title: '试验项目名称',
          align: 'center',
          dataIndex: 'unitName',
          customRender: (text) => {
            return text || '--'
          },
        },
      ],
      loadData: (params) => {
        let data = {
          ...params,
          ...this.queryParams,
        }
        return postAction(this.url.list, data).then((res) => {
          if (res.code == 200) {
            return res.data
          }
        })
      },
      selectedRowKeys: [],
      selectedRows: [],
    }
  },
  methods: {
    refresh(bool = true) {
      this.$refs.feeTable.refresh(bool)
    },
    show(selectedRows = [], selectedKeys = []) {
      if (isArray(selectedRows)) {
        this.selectedRows = selectedRows
      }
      if (isArray(selectedKeys)) {
        this.selectedRowKeys = selectedKeys
      }
      this.visible = true
    },
    handleOk() {
      this.$emit('change', this.selectedRowKeys, this.selectedRows)
      this.handleCancel()
    },
    handleCancel() {
      this.visible = false
    },
    onSelected(selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
    },
  },
}
</script>

<style>
</style>