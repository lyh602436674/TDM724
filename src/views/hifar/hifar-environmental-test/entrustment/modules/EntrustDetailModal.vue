<!--
 * @Author: 赵峰
 * @Date: 2021-09-09 11:14:48
 * @LastEditTime: 2021-11-18 13:56:29
 * @LastEditors: 马潭龙
 * @Descripttion: 报告详情
 * @FilePath: \hifar-platform-client\src\views\hifar\hifar-environmental-test\reports\modules\ReportDetailModal.vue
-->
<template>
  <h-modal
    :getContainer="getContainer"
    :visible="visible"
    destroyOnClose
    fullScreen
    inner
    title="委托详情"
    @cancel="handleCancel"
  >
    <div slot="footer" class="footer">
      <template v-if="detailData.status == 10">
        <a-button v-has="'entrustCheck:pass'" type="primary" @click="handleCheckPass(detailData.id,0)"> 审核通过</a-button>
        <a-button v-has="'entrustCheck:reject'" type="ghost-primary" @click="handleCheck(detailData)"> 审核驳回</a-button>
        <a-button v-has="'entrustCheck:epiboly'" type="ghost-primary" @click="handleEpiboly(detailData, '外包')"> 外包
        </a-button>
      </template>
      <a-button type="ghost-danger" @click="handleCancel"> 关闭</a-button>
    </div>
    <h-card :bordered="false">
      <h-tabs :activeKey="activeKey" :animated="true" @change="handleTabsChange">
        <a-tab-pane key="1" tab="委托信息">
          <entrust-detail ref='EntrustDetail' :detailData='detailData'></entrust-detail>
        </a-tab-pane>
        <a-tab-pane key="2" tab="委托单预览">
          <div class='autoHeight'>
            <embed
              v-if='detailData.reportPath'
              ref='iframe'
              :src='detailData.reportPath'
              frameborder='0'
              height='100%'
              scrolling='auto'
              width='100%'
            />
            <a-empty v-else style='margin-top: 160px'/>
          </div>
        </a-tab-pane>
      </h-tabs>
    </h-card>
  </h-modal>
</template>

<script>
import pdf from 'vue-pdf'
import {postAction} from '@/api/manage'
import EntrustDetail from "@views/hifar/hifar-environmental-test/entrustment/components/EntrustDetail";

export default {
  inject: {
    getContainer: {
      default: () => document.body,
    },
  },
  components: {
    EntrustDetail,
    pdf,
  },

  data() {
    return {
      visible: false,
      activeKey: '1',
      entrustId: '',
      detailData: {},
      model: {},
      type: '',
      url: {
        detail: '/HfEnvEntrustBusiness/queryById',
        check: '/HfEnvEntrustBusiness/submitConfirm',
        reject: '/HfEnvEntrustBusiness/reject',
        externalManage: '/HfEnvEntrustBusiness/externalManage',
      },
      numPages: 1, // pdf文件总页数
      pdfUrl: null,
      currentPage: 0,
      pageCount: 0,
      outsourcingUnit: "",
      testCondition: 2,
      testProgress: 2,
    }
  },
  methods: {
    show(record) {
      this.visible = true
      this.entrustId = record.id
      this.model = record
      this.loadDetail(record.id)
    },
    handleCancel() {
      this.visible = false
      this.$emit('change', true)
    },
    handleTabsChange(v) {
      this.activeKey = v
    },
    loadDetail(id) {
      let url = this.url.detail
      postAction(url, {id}).then((res) => {
        if (res.code === 200) {
          this.detailData = res.data
        }
      })
    },
    handleCheck(record, title) {
      this.$confirm({
        title: '提示',
        content: '确认驳回吗?',
        onOk: () => {
          postAction(this.url.reject, {id: record.id}).then(res => {
            if (res.code === 200) {
              this.$message.success('操作成功')
              this.handleCancel();
            } else {
              this.$message.warning(res.msg)
            }
          })
        },
      })
    },
    handleEpiboly(record) {
      this.$confirm({
        title: '提示:请填写外包单位',
        // content: '是否要申请外包?',
        content: h => <a-input v-model={this.outsourcingUnit} placeholder={"请输入外包单位"}><span
          slot="addonBefore">外包单位</span>
        </a-input>,
        onOk: () => {
          return new Promise((resolve, reject) => {
            if (!this.outsourcingUnit.replace(/\s*/g, "")) {
              this.$message.warning('请输入外包单位！')
              reject()
            } else {
              postAction(this.url.externalManage, {id: record.id, outsourcingUnit: this.outsourcingUnit}).then(res => {
                if (res.code === 200) {
                  this.$message.success('操作成功')
                  this.handleCancel();
                } else {
                  this.$message.warning(res.msg)
                }
              }).finally(() => {
                this.outsourcingUnit = ''
                resolve()
              })
            }
          })
        },
      })
    },
    handleCheckPass(id, isForce) {
      // let {entrustType} = this.detailData
      // this.$confirm({
      //   title: entrustType === '1' ? '提示' : '提示:请勾选试验条件和进度是否满足要求',
      //   // content: '是否要申请外包?',
      //   content: entrustType === '1' ? '确定审核通过吗' : h => {
      //     return (
      //       <div>
      //         <div>
      //           <span>试验条件:</span>
      //           <a-radio-group v-model={this.testCondition}>
      //             <a-radio-button value={1}>{'是'}</a-radio-button>
      //             <a-radio-button value={2}>{'是'}</a-radio-button>
      //           </a-radio-group>
      //         </div>
      //         <div>
      //           <span>进度:</span>
      //           <a-radio-group v-model={this.testProgress}>
      //             <a-radio-button value={1}>{'是'}</a-radio-button>
      //             <a-radio-button value={2}>{'是'}</a-radio-button>
      //           </a-radio-group>
      //         </div>
      //       </div>
      //     )
      //   },
      //   onOk: () => {
      //     console.log(this.testCondition, this.testProgress)
      //   }
      // })
      postAction(this.url.check, {id, isForce}).then((res) => {
        if (res.code === 200) {
          this.$message.success('操作成功')
          this.loadDetail(this.entrustId)
        } else {
          if (res.msg.includes('没有合适的设备')) {
            this.$confirm({
              title: '提示',
              content: '没有匹配到合适的设备，是否继续通过?',
              onOk: () => {
                postAction(this.url.check, {id, isForce: 1}).then(res => {
                  if (res.code === 200) {
                    this.$message.success('操作成功')
                    this.handleCancel();
                  } else {
                    this.$message.warning(res.msg)
                  }
                })
              },
            })
          }
        }
      })
    },
  },
}
</script>
<style lang='less' scoped>
iframe {
  position: absolute;
  top: 48px;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  margin: 0;
  padding: 0;
  border: 0;
}

.autoHeight {
  min-height: 700px;
  height: 800px;
}

/deep/ .ant-tabs-content {
  padding: 0 10px;
}
</style>