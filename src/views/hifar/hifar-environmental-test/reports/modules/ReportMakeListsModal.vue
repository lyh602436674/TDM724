<!--
 * @Author: 马潭龙
 * @Date: 2021-11-02 12:00:14
 * @LastEditors: 马潭龙
 * @Description: 按委托,按项目--弹窗
 * @LastEditTime: 2021-11-26 12:03:41
 * @FilePath: \hifar-platform-client\src\views\hifar\hifar-environmental-test\reports\modules\ReportMakeListsModal.vue
 -->
<template>
  <h-modal inner fullScreen destroyOnClose :visible='visible' :getContainer='getContainer' @cancel='handleCancel'>
    <a-alert v-if='messageErrorShow' :message='messageError' type='error' show-icon />
    <div class='footer' slot='footer'>
      <a-button type='ghost-danger' @click='handleCancel'> 关闭</a-button>
      <a-button v-if='custemkey == 1' :loading='submitLoading' @click='saveHandle' type='ghost-success'>
        保存
      </a-button>
      <a-button type='ghost-success' v-if='custemkey == 2' :loading='submitLoading' @click='subProjectHandle'>
        保存
      </a-button>
      <a-button type='ghost-success' v-if='custemkey == 3' :loading='submitLoading' @click='subProjectHandle'>
        保存
      </a-button>
      <a-button type='ghost-success' v-if='custemkey == 4' :loading='submitLoading' @click='saveTestHandle'>
        保存
      </a-button>
    </div>
    <h-card fixed>
      <a-tabs v-model='custemkey' style='width: 100%'>
        <!--     <a-tab-pane :key='1' tab='按项目查找' />
           <a-tab-pane :key="2" tab="按委托查找" />
                <a-tab-pane :key="3" tab="按试品查找" />-->
                <a-tab-pane :key="4" tab="按试验查找" />
      </a-tabs>
      <template v-if='custemkey == 2'>
        <report-custem-tree ref='ReportStandOut'></report-custem-tree>
      </template>
      <template v-if='custemkey == 1'>
        <report-entrust-table ref='reportEntrustTable' @change='selectChange'></report-entrust-table>
      </template>
      <template v-if='custemkey == 3'>
        <Report-Product-Main ref='ReportProductMain'></Report-Product-Main>
      </template>
      <template v-if='custemkey == 4'>
        <report-experiment-table ref='reportExperimentMain' @change='selectChange'></report-experiment-table>
      </template>
    </h-card>
    <report-cover-modal ref='ReportCoverModal' @change='selectCoverHandle'></report-cover-modal>
  </h-modal>
</template>

<script>
import ReportCustemTree from '../components/ReportCustemTree.vue'
import ReportStandOut from '../components/ReportStandOut'
import ReportCoverModal from '../modules/ReportCoverModal'
import { postAction } from '@/api/manage'
import ReportProductMain from '../components/ReportProductMain.vue'
// import ReportExperimentMain from '../components/ReportExperimentMain.vue'
// import ReportEntrustMain from '../components/ReportEntrustMain.vue'
import ReportEntrustTable from '../components/ReportEntrustTable.vue'
import ReportExperimentTable from '../components/ReportExperimentTable'

export default {
  inject: {
    getContainer: {
      default: () => document.body
    }
  },
  data() {
    return {
      visible: false,
      custemkey: 4,
      messageError: '',
      submitLoading: false,
      messageErrorShow: false,
      selectedRowKeys: [],
      selectedRow: [],
      entrustCodeArr: [],
      url: {
        submit: '/HfEnvReportBusiness/add'
      }
    }
  },
  components: {
    ReportCustemTree,
    ReportCoverModal,
    ReportStandOut,
    ReportProductMain,
    // ReportExperimentMain,
    // ReportEntrustMain,
    ReportEntrustTable,
    ReportExperimentTable
  },
  methods: {
    openNotificationWithIcon(type, desc) {
      this.$notification[type]({
        duration: 2,
        message: '此功能正在开发中',
        description: desc
      })
    },
    openNotificationWithIconb(type, desc) {
      this.$notification[type]({
        duration: 2,
        message: '请至少选择一项',
        description: desc
      })
    },
    openNotificationWithIconc(type, desc) {
      this.$notification[type]({
        duration: 2,
        message: '请选择项目',
        description: desc
      })
    },
    // 关闭
    handleCancel() {
      this.visible = false
      this.selectedRow = []
      this.selectedRowKeys = []
    },
    show(record) {
      this.visible = true
      this.custemkey = 4
    },
    cloes() {
      this.visible = false
    },
    // 委托单页面保存
    subProjectHandle(record) {
      this.openNotificationWithIcon('error')
    },
    // 项目页面保存
    selectChange(selectedRowKeys, selectedRow) {
      let entrustCodeArr = []
      if (selectedRow.length > 0) {
        selectedRow.forEach((item) => {
          entrustCodeArr.push(item.entrustCode)
        })
      }
      this.entrustCodeArr = entrustCodeArr
      this.selectedRow = selectedRow
      this.selectedRowKeys = selectedRowKeys
      this.$emit('change')
    },
    // 按项目查找生成报告按钮
    saveHandle() {
      let { entrustCodeArr, selectedRowKeys } = this
      if (selectedRowKeys.length == 0) {
        this.openNotificationWithIconc('error', '请选择项目')
      } else {
        // 检验是否总号一样
        // let sumMarkArr = this.selectedRow.map(item => item.sumMark)
        // let sumMarkSet = new Set(sumMarkArr)
        // if (sumMarkSet.size > 1) {
        //   this.openNotificationWithIconc('error', '请选择相同的总号项目')
        //   return
        // }

        // 选择封面注释
        // this.$refs.ReportCoverModal.visible = true
        let projectId = []
        let selectedRow = this.selectedRow
        selectedRow.forEach((item) => {
          projectId.push(item.id)
        })
        this.submitLoading = true
        let data = {
          projects: selectedRow
        }
        let params = {
          ...data,
          buttonFlag: 'save'
        }
        postAction('/HfEnvReportBusiness/add', params).then((res) => {
          this.submitLoading = false
          if (res.code == 200) {
            this.$message.success('操作成功')
            this.$emit('change', true)
            this.handleCancel()
          }
        })
      }
    },
    // 按试验生成报告按钮
    saveTestHandle() {
      let { entrustCodeArr, selectedRowKeys } = this
      if (selectedRowKeys.length == 0) {
        this.openNotificationWithIconc('error', '请选择试验')
      } else {
        // 检验是否总号一样
        // let sumMarkArr = this.selectedRow.map(item => item.sumMark)
        // let sumMarkSet = new Set(sumMarkArr)
        // if (sumMarkSet.size > 1) {
        //   this.openNotificationWithIconc('error', '请选择相同的总号项目')
        //   return
        // }

        // 选择封面注释
        // this.$refs.ReportCoverModal.visible = true
        let testId = []
        let selectedRow = this.selectedRow
        selectedRow.forEach((item) => {
          testId.push(item.id)
        })
        this.submitLoading = true
        let data = {
          tests: selectedRow
        }
        let params = {
          ...data,
          buttonFlag: 'save'
        }
        postAction('/HfEnvReportBusiness/addByTest', params).then((res) => {
          this.submitLoading = false
          if (res.code == 200) {
            this.$message.success('操作成功')
            this.$emit('change', true)
            this.handleCancel()
          }
        })
      }
    },

    selectCoverHandle(v) {
      let projectId = []
      let selectedRow = this.selectedRow
      let coverTemplateId = v.coverTemplateId
      selectedRow.forEach((item) => {
        projectId.push(item.id)
      })
      if (coverTemplateId) {
        this.submitLoading = true
        let data = {
          entrustId: selectedRow[0].entrustId,
          entrustCode: selectedRow[0].entrustCode,
          custId: selectedRow[0].custId,
          custName: selectedRow[0].custName,
          custLinkName: selectedRow[0].custLinkName,
          custLinkMobile: selectedRow[0].custLinkMobile,
          productId: selectedRow[0].productId,
          productName: selectedRow[0].productName,
          coverTemplateId: coverTemplateId,
          projectId: projectId.length > 0 ? projectId.join(',') : '',
          unitId: selectedRow[0].unitId
        }
        let params = {
          ...data,
          buttonFlag: 'save'
        }
        postAction('/HfEnvReportBusiness/add', params).then((res) => {
          this.submitLoading = false
          if (res.code == 200) {
            this.$message.success('操作成功')
            this.$emit('change', true)
            this.handleCancel()
          }
        })
      }
    }
  }
}
</script>

<style lang='less' scoped>
</style>