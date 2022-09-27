<!--
 * @Author: 赵峰
 * @Date: 2021-09-15 10:09:09
 * @LastEditTime: 2021-12-06 21:48:37
 * @LastEditors: 赵文凯
 * @Descripttion: 委托管理--委托列表
 * @FilePath: \tdm200-client\src\views\hifar\hifar-environmental-test\entrustment\EntrustmentsList.vue
-->
<template>
  <div ref='entrustmentList' class='entrustment-list'>
    <h-card fixed>
      <template slot='title'> 委托管理</template>
      <h-search slot='search-form' v-model='queryParams' :data='searchForm' size='default' @change='refresh(true)'/>
      <template slot='table-operator'>
        <a-button v-has="'entrustment:signAndIssue'" icon='solution' size='small' type='ghost-primary'
                  :loading="signAndIssueLoading"
                  @click='handleSignAndIssue'>{{ signAndIssueLoading ? '签发中' : '签发' }}
        </a-button>
        <a-button v-has="'entrustment:dataEntry'" icon='qrcode' size='small' type='ghost-primary'
                  @click='handleDataEntry'>扫码创建
        </a-button>
        <template v-has="'entrustment:add'">
          <a-button icon='plus' size='small' type='ghost-primary' @click='handleAdd'> 添加</a-button>
        </template>
        <a-button icon="download" size="small" type="ghost-warning" @click="handleExportXls('委托单信息')">
          导出
        </a-button>
        <template v-if='selectedRowKeys.length>0'>
          <a-button v-has="'entrustment:delete'" icon='delete' size='small' type='danger' @click='batchDel'>
            批量删除
          </a-button>
        </template>
      </template>
      <h-vex-table
        ref='entrustmentListTable'
        slot='content'
        :columns='columns'
        :data='loadData'
        :rowSelection='{ selectedRowKeys: selectedRowKeys, onSelect: onSelect }'
      >
        <template #isExternalManage="text">
          <span :style="{color:text === '1' ? 'red':text === '0'?'green':'black' }">{{
              text === '1' ? '是' : text === '0' ? '否' : '--'
            }}</span>
        </template>
        <span slot='entrustCode' slot-scope='text, record'>
          <h-icon v-if="record.entrustType == '1'" type='icon-nei'/>
          <h-icon v-else type='icon-wai'/>
          <a href='javascript:;' @click='handleDetailCode(record)'>
            {{ record.entrustCode ? record.entrustCode : '--' }}</a
          >
        </span>
        <span slot='status' slot-scope='text, record'>
          <a-badge :color='record.status | wtStatusColorFilter' :text='record.status | wtStatusFilter'/>
        </span>
        <template slot='actions' slot-scope='text, record'>
          <a-tooltip title='详情'>
            <a-icon
              class='primary-text'
              style='cursor: pointer'
              title='详情'
              type='eye'
              @click='() => handleDetail(record)'
            />
          </a-tooltip>
          <a-divider type='vertical'/>
          <a-dropdown>
            <a-tooltip title='更多'>
              <a class='ant-dropdown-link'>
                <h-icon type='icon-gengduo1'/>
              </a>
            </a-tooltip>
            <a-menu slot='overlay'>
              <a-menu-item v-if='record.status == 1 || record.status == 30'>
                <a @click='handleEdit(record)'>编辑</a>
              </a-menu-item>
              <a-menu-item>
                <a @click='handleCopyItem(record)'>复制</a>
              </a-menu-item>
              <a-menu-item v-if='record.status == 1'>
                <a-popconfirm v-has="'entrustment:delete'" title='确定删除吗?' @confirm='() => handleDelete(record.id)'>
                  <a style='color: #ff4d4f'>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </template>
      </h-vex-table>
    </h-card>
    <entrustment-inner-modal ref="EntrustmentModal" @change='refresh(true)'></entrustment-inner-modal>
    <entrustment-detail-modal ref='EntrustmentDetailModal'></entrustment-detail-modal>
    <task-submit-modal ref='TaskSubmitModal' @change='refresh(true)'></task-submit-modal>
    <a-alert v-if='messageErrorShow' :message='messageError' show-icon type='error'/>
    <entrust-data-entry-modal ref="EntrustDataEntryModal"></entrust-data-entry-modal>
  </div>
</template>

<script>
import moment from 'moment'
import {downloadFile, postAction} from '@/api/manage'
import mixin from '@/views/hifar/hifar-environmental-test/mixin.js'
import EntrustmentDetailModal from './modules/EntrustmentDetailModal'
import TaskSubmitModal from './modules/TaskSubmitModal'
import EntrustDataEntryModal from '@/views/hifar/hifar-environmental-test/entrustment/modules/EntrustDataEntryModal'
import EntrustmentInnerModal from "@views/hifar/hifar-environmental-test/entrustment/modules/EntrustmentInnerModal";

export default {
  provide() {
    return {
      getContainer: () => this.$refs.entrustmentList
    }
  },
  components: {
    EntrustmentInnerModal,
    EntrustmentDetailModal,
    TaskSubmitModal,
    EntrustDataEntryModal
  },
  mixins: [mixin],
  data() {
    return {
      moment,
      url: {
        list: '/HfEnvEntrustBusiness/listPage',
        delete: '/HfEnvEntrustBusiness/logicRemoveById',
        copy: '/HfEnvEntrustBusiness/copyById',
        flowNode: '/FlowBusiness/listStartNextData',
        submitUrl: '/HfEnvEntrustBusiness/submit',
        submitConfirm: '/HfEnvEntrustBusiness/submitConfirm',
        batchSubmitConfirm: '/HfEnvEntrustBusiness/batchSubmitConfirm',
        export: '/HfEnvEntrustBusiness/listAllForExport',
        signAndIssue: "/HfEnvEntrustBusiness/signAndIssue"
      },
      queryParams: {},
      selectedRowKeys: [],
      selectedRows: [],
      searchForm: [
        {
          title: '委托单号',
          key: 'c_entrustCode_7',
          formType: 'input'
        },
        {
          title: '送试单位',
          key: 'c_custName_7',
          formType: 'input'
        },
        {
          title: '试品名称',
          key: 'productName',
          formType: 'input'
        },
        {
          title: "试品工号",
          key: "productCode",
          formType: 'input'
        },
        {
          title: '试品代号',
          key: 'productAlias',
          formType: 'input'
        },
        {
          title: '试品编号',
          key: 'pieceNo',
          formType: 'input'
        },
        {
          title: '试验项目',
          key: 'unitName',
          formType: 'input'
        },
        {
          title: '状态',
          key: 'c_status_1',
          formType: 'select',
          options: [
            {
              title: '草稿',
              key: 1,
              value: 1
            },
            {
              title: '已提交',
              key: 10,
              value: 10
            },
            {
              title: '已通过',
              key: 20,
              value: 20
            },
            {
              title: '已驳回',
              key: 30,
              value: 30
            },
            {
              title: '待出报告',
              key: 40,
              value: 40
            },
            {
              title: '已出报告',
              key: 50,
              value: 50
            }
          ]
        },
        {
          title: '是否外包',
          key: 'c_isExternalManage_1',
          formType: 'select',
          options: [
            {
              title: '是',
              key: 1,
              value: 1
            },
            {
              title: '否',
              key: 0,
              value: 0
            }
          ]
        },
        {
          title: '外包单位',
          key: 'c_outsourcingUnit_7',
          formType: 'input'
        },

      ],
      columns: [
        {
          title: '委托单号',
          align: 'left',
          width: 140,
          dataIndex: 'entrustCode',
          scopedSlots: {customRender: 'entrustCode'},
          fixed: 'left'
        },
        {
          title: '状态',
          align: 'left',
          dataIndex: 'status',
          minWidth: 100,
          scopedSlots: {customRender: 'status'}
        },
        {
          title: '是否外包',
          align: 'left',
          dataIndex: 'isExternalManage',
          minWidth: 100,
          scopedSlots: {customRender: 'isExternalManage'},
        },
        {
          title: '外包单位',
          align: 'left',
          dataIndex: 'outsourcingUnit',
          minWidth: 180,
          customRender: (text, record) => {
            return text || '--'
          },
        },
        {
          title: '送试单位',
          align: 'left',
          dataIndex: 'custName',
          minWidth: 100,
          customRender: (text, record) => {
            return text || '--'
          }
        },
        {
          title: '签发人',
          align: 'left',
          dataIndex: 'signAndIssuePerson',
          minWidth: 100,
          customRender: (text, record) => {
            return text || '--'
          }
        },
        {
          title: '分号',
          align: 'left',
          dataIndex: 'separateMarks',
          minWidth: 100,
          customRender: (text, record) => {
            return text || '--'
          }
        },
        {
          title: '总号',
          align: 'left',
          dataIndex: 'sumMarks',
          minWidth: 100,
          customRender: (text, record) => {
            return text || '--'
          }
        },
        {
          title: '试品名称',
          align: 'left',
          minWidth: 200,
          dataIndex: 'productNames',
          customRender: (text, record) => {
            return text || '--'
          }
        },
        {
          title: "试品代号",
          align: "left",
          dataIndex: "productAliass",
          minWidth: 100,
          customRender: (text, record) => {
            return text || "--";
          },
        },
        {
          title: "试品工号",
          minWidth: 150,
          dataIndex: "productCodes",
          align: "center",
          customRender: (t) => {
            return t || '--'
          }
        },
        {
          title: "试品编号",
          align: "left",
          dataIndex: "pieceNo",
          minWidth: 100,
          customRender: (text, record) => {
            return text || "--";
          },
        },
        {
          title: '试品数量',
          align: 'center',
          dataIndex: 'productNums',
          minWidth: 100,
          customRender: (text, record) => {
            return text || '--'
          }
        },
        {
          title: "联系人",
          align: "center",
          dataIndex: "linkName",
          width:120,
          customRender: (text, record) => {
            return text || "--";
          },
        },
        {
          title: "联系方式",
          align: "center",
          width:150,
          dataIndex: "linkMobile",
          customRender: (text, record) => {
            return text || "--";
          },
        },
        {
          title: '试验性质',
          align: 'left',
          minWidth: 80,
          dataIndex: 'testPropertyCodeText',
          customRender: (text, record) => {
            return text || "--";
          }
        },
        {
          title: '试验项目',
          align: 'left',
          dataIndex: 'unitNames',
          minWidth: 100,
          customRender: (text, record) => {
            return text || '--'
          }
        },
        {
          title: '委托日期',
          align: 'left',
          dataIndex: 'entrustTime',
          minWidth: 100,
          customRender: (time) => {
            return time && time != 0 ? moment(parseInt(time)).format('YYYY-MM-DD') : '--'
          }
        },
        {
          title: '创建人 ',
          align: 'left',
          minWidth: 100,
          dataIndex: 'createUserName',
          customRender: (text, record) => {
            return text || '--'
          }
        },
        {
          title: '创建时间 ',
          align: 'left',
          minWidth: 140,
          dataIndex: 'createTime',
          customRender: (text, record) => {
            return text && text != 0 ? moment(parseInt(text)).format('YYYY-MM-DD HH:mm:ss') : '--'
          }
        },
        {
          title: '操作',
          dataIndex: 'actions',
          fixed: 'right',
          width: 100,
          align: 'center',
          scopedSlots: {customRender: 'actions'}
        }
      ],
      loadData: (params) => {
        let data = {
          ...params,
          ...this.queryParams,
          entrustType: '1'
        }
        return postAction(this.url.list, data).then((res) => {
          if (res.code === 200) {
            return res.data
          }
        })
      },
      signAndIssueLoading: false,
    }
  },
  methods: {
    refresh(bool = true) {
      this.$refs.entrustmentListTable.refresh(bool)
      this.selectedRowKeys = []
      this.selectedRows = []
    },
    onSelect(selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
    },
    handleDataEntry() {
      this.$refs.EntrustDataEntryModal.open()
    },
    handleAdd() {
      let record = {}
      let type = 'add'
      this.$refs.EntrustmentModal.show(record, type)
    },
    async handleExportXls(name, model) {
      let data = {
        ...this.queryParams,
        ...model,
        ids: this.selectedRowKeys.join(','),
        type: "export"
      }
      let url = this.url.list
      let params = data
      let fileName = name + '.xls'
      await downloadFile(url, fileName, params)
    },
    submitConfirm(record) {
      let _this = this
      postAction(_this.url.submitConfirm, {id: record.id}).then((res) => {
        if (res.code === 200) {
          _this.$message.success('确认成功')
          _this.refresh()
        } else if (res.code === 502) {
          this.$confirm({
            title: '是否外包',
            content: '没有合适的设备,是否外包?',
            okText: '外包',
            onOk: function () {
              postAction(_this.url.submitConfirm, {id: record.id, isExternalManage: 1}).then((response) => {
                if (response.code === 200) {
                  _this.$message.success('外包成功')
                  _this.refresh()
                } else {
                  _this.$message.success(response.msg)
                }
              })
            }
          })
        } else {
          _this.$message.success(res.msg)
        }
      })
    },
    // 复制
    handleCopyItem(record) {
      let url = this.url.copy
      postAction(url, {id: record.id}).then((res) => {
        if (res.code == 200) {
          this.$message.success('复制成功!')
          this.refresh(true)
        }
      })
    },
    // 详情
    handleDetail(record) {
      this.$refs.EntrustmentDetailModal.show(record.id)
    },
    // 编辑
    handleEdit(record) {
      let type = 'editor'
      this.$refs.EntrustmentModal.show(record, type)
    },
    // 删除
    handleDelete(id) {
      postAction(this.url.delete, {id: id}).then((res) => {
        if (res.code === 200) {
          this.$message.success('删除成功')
          this.refresh(true)
        }
      })
    },
    //签发
    handleSignAndIssue() {
      if (this.selectedRowKeys.length) {
        this.$confirm({
          title: '提示',
          content: '确认签发吗？',
          onOk: () => {
            this.signAndIssueLoading = true
            postAction(this.url.signAndIssue, {ids: this.selectedRowKeys.toString()}).then((res) => {
              if (res.code === 200) {
                this.$message.success('签发成功')
                this.refresh()
                this.signAndIssueLoading = false
              }
            })
          }
        })
      } else {
        this.$message.warning('请至少选择一项')
      }
    },
    // 批量删除
    batchDel() {
      let _this = this
      if (_this.selectedRowKeys.length) {
        this.$confirm({
          title: '确认删除',
          content: '删除后不可恢复，确认删除？',
          onOk: function () {
            postAction(_this.url.delete, {id: _this.selectedRowKeys.join()}).then((res) => {
              if (res.code === 200 && res.data.num) {
                _this.$message.success('删除成功')
                _this.refresh()
              }
            })
          }
        })
      }
    },
    //1 草稿 10 提交 20审核通过 30审核驳回 40检测完成 50 已出报告 80终止 99逻辑删除
    handleDetailCode(record) {
      if (record.status == 1 || record.status == 30) {
        this.handleEdit(record)
      } else if (record.status >= 10 && record.status != 30) {
        this.handleDetail(record)
      }
    },
    getCheckboxProps(record) {
      return record.status == 1 || record.status == 30
    },
  }
}
</script>
<style lang='less' scoped>
.entrustment-list {
  width: 100%;
  height: 100%;
  position: relative;
}
</style>