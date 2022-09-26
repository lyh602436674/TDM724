<!--
 * @Author: 陈乾龙
 * @Date: 2021-09-29 14:29:41
 * @LastEditTime: 2021-11-22 15:54:12
 * @LastEditors: 马潭龙
 * @Description: 任务详情
 * @FilePath: \hifar-platform-client\src\views\hifar\hifar-environmental-test\task\modules\TaskDetail.vue
-->
<template>
  <div>
    <h-modal
      :fullScreen="fullScreen"
      :getContainer="getContainer"
      :visible="visible"
      title="委托任务详情"
      width="90%"
      @cancel="handleCancel"
    >
      <div class="fullscreenIcon" @click="fullScreenHandle">
        <a-icon :type="!fullScreen ? 'fullscreen' : 'fullscreen-exit'" class="primary-text" style="font-size: 16px"/>
      </div>
      <div class="task-detail-wrapper">
        <div class="task-info">
          <h-desc :data="model" labelWidth="130px" size="small" title="基本信息">
            <h-desc-item label="委托单号">
              {{ model.entrustCode }}
              <a-button icon="eye" size="small" style="margin-left: 10px" type="primary" @click="entrustReview">委托单预览
              </a-button>
            </h-desc-item>
            <h-desc-item label='委托状态'>
              <a-tag slot='content' :color='model.entrustData.status | wtStatusColorFilter' class='status_tag'>
                {{ model.entrustData.status | wtStatusFilter }}
              </a-tag>
            </h-desc-item>
            <h-desc-item label='委托日期'>
              {{
                model.entrustData.entrustTime && model.entrustData.entrustTime != 0
                  ? moment(parseInt(model.entrustData.entrustTime)).format('YYYY-MM-DD')
                  : '--'
              }}
            </h-desc-item>
            <h-desc-item label='委托单类型'>
              {{ model.entrustData.entrustType | entrustTypeFilter }}
            </h-desc-item>
            <h-desc-item label="送试单位">
              {{ model.custName }}
            </h-desc-item>
            <h-desc-item label="联系人">
              {{ model.linkName }}
            </h-desc-item>
            <h-desc-item label="联系方式">
              {{ model.linkMobile }}
            </h-desc-item>
            <h-desc-item label='客户地址' v-if="model.entrustData.entrustType == 2">
              {{ model.entrustData.custAddress ? model.entrustData.custAddress : '--' }}
            </h-desc-item>
            <h-desc-item label='任务编码'>
              {{ model.outSourceCode ? model.outSourceCode : '--' }}
            </h-desc-item>
            <h-desc-item label='试验性质'>
              {{ model.entrustData.testPropertyName ? model.entrustData.testPropertyName : '--' }}
            </h-desc-item>
            <h-desc-item label='密级'>
              {{ model.entrustData.secretLevelCode_dictText ? model.entrustData.secretLevelCode_dictText : '--' }}
            </h-desc-item>
            <h-desc-item label='是否外包'>
              <span v-if="model.entrustData.isExternalManage == 1" slot="content" style="color:red">是</span>
              <span v-else slot="content" style="color:green">否</span>
            </h-desc-item>
            <h-desc-item label='是否出报告'>
              {{ model.entrustData.isReport == 1 ? '是' : model.entrustData.isReport == 2 ? '否' : '--' }}
            </h-desc-item>
            <h-desc-item :label='model.entrustData.entrustType === "1" ? "委托方批准人" : "委托方代表"'>
              {{ model.entrustData.entrustApproval ? model.entrustData.entrustApproval : '--' }}
            </h-desc-item>
            <template v-if="model.entrustData.entrustType == 2">
              <h-desc-item label='是否允许对试品进行拍照'>
                {{ model.entrustData.isPhotographByPiece == 1 ? '是' : model.entrustData.isPhotographByPiece == 2 ? '否' : '--' }}
              </h-desc-item>
              <h-desc-item label='是否需要附试验原始数据'>
                {{ model.entrustData.isNeedOriginalData == 1 ? '是' : model.entrustData.isNeedOriginalData == 2 ? '否' : '--' }}
              </h-desc-item>
            </template>
            <h-desc-item v-if="model.entrustData.entrustType==='2'" label='报告交付进度要求'>
              {{
                model.entrustData.progressRequire_dictText ? model.entrustData.progressRequire_dictText : '--'
              }}
            </h-desc-item>
            <h-desc-item v-if="model.entrustData.entrustType==='2'" label='结算方式'>
              {{
                model.entrustData.settlementType_dictText ? model.entrustData.settlementType_dictText : '--'
              }}
            </h-desc-item>
            <h-desc-item label='创建人'>
              {{ model.entrustData.createUserName ? model.entrustData.createUserName : '--' }}
            </h-desc-item>
            <h-desc-item  label='创建时间'>
              {{
                model.entrustData.createTime && model.entrustData.createTime != 0
                  ? moment(parseInt(model.entrustData.createTime)).format('YYYY-MM-DD HH:mm:ss')
                  : '--'
              }}
            </h-desc-item>
            <h-desc-item :span='4' label='委托单附件'>
              <div slot='content'>
                <template v-if='model.entrustData.attachInfo && model.entrustData.attachInfo.length'>
                  <div v-for='(item, index) in model.entrustData.attachInfo' :key='index' class='url-list'>
                    <span>{{ index + 1 }}、{{ item.fileName }}</span>
                    <a-button icon='download' size='small' type='primary' style="margin-left: 10px"
                              @click='handleDownload(item.filePath, item.fileName)'>
                      下载
                    </a-button>
                  </div>
                </template>
                <span v-else>暂无附件</span>
              </div>
            </h-desc-item>

            <h-desc-item :span="3" label="备注">
              {{ model.entrustData.remarks }}
            </h-desc-item>
          </h-desc>
        </div>
        <div class="piece-info">
          <h-desc labelWidth="120px" title="试品信息">
            <a-table
              :columns="pieceColumns[+model.entrustData.entrustType]"
              :dataSource="pieceInfo"
              :pagination="false"
              bordered
              rowKey="id"
              size="small"
              style="width: 100%; height: 100%"
            ></a-table>
          </h-desc>
        </div>

        <div class="piece-info">
          <h-desc labelWidth="130px" size="small" title="试验项目">
            <h-desc-item label="项目名称">
              {{ model.unitName ? model.unitName : '--' }}
            </h-desc-item>
            <h-desc-item label="试验名称">
              {{ model.testName ? model.testName : '--' }}
            </h-desc-item>
            <h-desc-item label="试验标准">
              {{ model.standardCode + model.standardName || '--' }}
            </h-desc-item>
<!--            <h-desc-item label="预计开始时间">-->
<!--              {{-->
<!--                model.expectStartTime && model.expectStartTime != 0-->
<!--                  ? moment(parseInt(model.expectStartTime)).format('YYYY-MM-DD HH:mm')-->
<!--                  : '&#45;&#45;'-->
<!--              }}-->
<!--            </h-desc-item>-->
            <h-desc-item label="预计时长（h）">
              {{ model.predictDuration ? model.predictDuration + '小时' : '--' }}
            </h-desc-item>
<!--            <h-desc-item label="试验部门">-->
<!--              {{ model.workName ? model.workName : '&#45;&#45;' }}-->
<!--            </h-desc-item>-->
            <h-desc-item label="总号">
              {{ model.sumMark ? model.sumMark : '--' }}
            </h-desc-item>
            <h-desc-item label="分号">
              {{ model.separateMark ? model.separateMark : '--' }}
            </h-desc-item>
            <h-desc-item v-if="model.needProcess == 1" :span="2" label="加工要求">
              {{ model.processRequire ? model.processRequire : '--' }}
            </h-desc-item>
            <h-desc-item :span="4" label="试验依据">
              {{ model.checkRequire ? model.checkRequire : '--' }}
            </h-desc-item>
            <h-desc-item :span="4" label="其他条件">
              {{ model.testCondition ? model.testCondition : '--' }}
            </h-desc-item>
            <h-desc-item :span='4' label='附件'>
              <div slot='content'>
                <template v-if='model.fileInfo && model.fileInfo.length'>
                  <div v-for='(item, index) in model.fileInfo' :key='index' class='url-list'>
                    <span>{{ index + 1 }}、{{ item.fileName }}</span>
                    <a-button icon='download' size='small' type='primary' style="margin-left: 10px"
                              @click='handleDownload(item.filePath, item.fileName)'>
                      下载
                    </a-button>
                  </div>
                </template>
                <span v-else>暂无附件</span>
              </div>
            </h-desc-item>
          </h-desc>
        </div>
        <div v-if="model.curveUrl" class="piece-info">
          <h-desc labelWidth="120px" size="small" title="规划曲线">
            <img :src="model.curveUrl" alt="规划曲线" height="90%" width="90%">
          </h-desc>
        </div>
        <div class="piece-info">
          <h-desc labelWidth="120px" title="试验条件要求">
            <template v-if="!abilityRequire.length"><a-empty style="width: 100%"></a-empty></template>
            <a-tabs v-else :default-active-key="0" style="width: 100%">
              <template v-for="(proItem,itemIndex) in abilityRequire">
                <a-tab-pane :key="itemIndex"
                            :tab="proItem.type === 'stage' ? proItem.title + itemIndex : proItem.title">
                  <vxe-table
                    :key="itemIndex + '-only'"
                    :auto-resize="true"
                    :data="proItem.abilityInfo"
                    border
                    highlight-hover-row
                    keep-source
                    size="mini"
                  >
                    <vxe-table-column type="seq" width="60"></vxe-table-column>
                    <vxe-table-column field="paramName" title="设备能力"></vxe-table-column>
                    <vxe-table-column field="unitName" title="单位"></vxe-table-column>
                    <vxe-table-column field="curveType" title="曲线类型">
                      <template #default="{ row }">
                        <span>{{ row.curveType === '1' ? '温度/℃' : row.curveType === '2' ? '湿度/RH' : '--' }}</span>
                      </template>
                    </vxe-table-column>
                    <vxe-table-column field="conditionTypeDesc" title="条件">
                      <template #default="{ row }">
                      <span v-if="row.paramName === '初始类型'">{{
                          row.conditionTypeDesc === '1' ? '先高温' : row.curveType === '2' ? '先低温' : '--'
                        }}</span>
                        <span v-else>{{ row.conditionTypeDesc }}</span>
                      </template>
                    </vxe-table-column>
                  </vxe-table>
                </a-tab-pane>
              </template>
            </a-tabs>
          </h-desc>
        </div>

        <div class="equip-info">
          <h-desc title="试验信息">
            <a-table
              :columns="equipColumns"
              :dataSource="equipTestInfo"
              :pagination="false"
              bordered
              rowKey="id"
              size="small"
              style="width: 100%; height: 100%"
            >
              <a-icon
                slot="record"
                slot-scope="text, record"
                class="primary-text"
                type="carry-out"
                @click="showRecord(record)"
              />
            </a-table>
          </h-desc>
        </div>
      </div>
      <div slot="footer">
        <a-button type="ghost-danger" @click="handleCancel">关闭</a-button>
      </div>
    </h-modal>
    <pieces-record ref="piecesRecord"/>
    <test-entrust-review-pdf ref="testEntrustReviewPdf"/>
  </div>
</template>

<script>
import {downloadFile, getFileAccessHttpUrl, postAction} from '@/api/manage'
import moment from 'moment'
import PiecesRecord from './PiecesRecord'
import mixin from '@/views/hifar/mixin.js'
import TestEntrustReviewPdf from "@views/hifar/hifar-environmental-test/task/modules/TestEntrustReviewPdf";

export default {
  mixins: [mixin],
  provide() {
    return {
      getContainer: () => document.body,
    }
  },
  inject: {
    getContainer: {
      default: () => {
        return () => document.body
      },
    },
  },
  components: {
    // PiecesRecordHDesc,
    PiecesRecord, TestEntrustReviewPdf
  },
  data() {
    return {
      moment,
      title: '--',
      visible: false,
      fullScreen: false,
      model: {},
      pieceInfo: [],
      abilityRequire: [],
      equipTestInfo: [],
      url: {
        detail: '/HfEnvTaskBusiness/queryById',
      },
      pieceColumns: {
        1: [
          {
            title: '试品名称',
            dataIndex: 'productName',
          },
          {
            title: '试品工号',
            dataIndex: 'productCode',
          },
          {
            title: '试品代号',
            dataIndex: 'productAlias',
          },
          {
            title: '试品编号',
            dataIndex: 'pieceNo'
          }
        ],
        2: [
          {
            title: '试品名称',
            dataIndex: 'productName',
          },
          {
            title: '试品代号',
            dataIndex: 'productAlias',
          },
          {
            title: '试品数量',
            dataIndex: 'pieceNum',
          },
          {
            title: '试品编号',
            dataIndex: 'pieceNo'
          }
        ],
      },
      equipColumns: [
        {
          title: '设备名称',
          dataIndex: 'equipName',
        },
        {
          title: '设备编号',
          dataIndex: 'equipCode',
        },
        {
          title: '设备类型',
          dataIndex: 'equipTypeName',
        },
        {
          title: '内部名称',
          dataIndex: 'innerName',
        },
        {
          title: '实际开始时间',
          dataIndex: 'realStartTime',
          customRender(text) {
            return text != 0 ? moment(parseFloat(text)).format('YYYY-MM-DD HH:mm:ss') : '--'
          },
        },
        {
          title: '实际结束时间',
          dataIndex: 'realEndTime',
          customRender(text) {
            return text != 0 ? moment(parseFloat(text)).format('YYYY-MM-DD HH:mm:ss') : '--'
          },
        },
        {
          title: '流转记录',
          dataIndex: 'record',
          fixed: 'right',
          align: 'center',
          width: 80,
          scopedSlots: {
            customRender: 'record',
          },
        },
      ],
    }
  },
  methods: {
    show(record = {}) {
      this.model = Object.assign({}, record)
      this.title = record.taskCode + '详情'
      this.getTaskDetail()
      this.$nextTick(() => {
        this.visible = true
      })
    },
    handleDownload(filePath, fileName) {
      let fileAccessUrl = getFileAccessHttpUrl(filePath)
      downloadFile(fileAccessUrl, fileName)
    },
    entrustReview() {
      this.$refs.testEntrustReviewPdf.show(this.model.entrustData.reportPath)
    },
    getTaskDetail() {
      let params = {
        id: this.model.id,
      }
      postAction(this.url.detail, params).then((res) => {
        if (res.code == 200) {
          this.model = Object.assign({}, this.model, res.data)
          this.pieceInfo = res.data.pieceInfo || []
          this.equipTestInfo = res.data.equipTestInfo || []
          this.abilityRequire = res.data.abilityRequire || []
        }
      })
    },
    showRecord(record) {
      this.$refs.piecesRecord.show(record)
    },
    handleCancel() {
      this.visible = false
      this.fullScreen = false
    },
    fullScreenHandle() {
      this.fullScreen = !this.fullScreen
    },
  },
}
</script>

<style lang="less">
.task-detail-wrapper {
  height: 100%;
  padding: 10px;
  overflow: auto;

  .task-info,
  .piece-info,
  .equip-info {
    margin-bottom: 10px;

    .ant-card-body {
      padding: 10px 0;
    }
  }
}
</style>
<style lang="less" scoped>
/deep/ .ant-table-small > .ant-table-content > .ant-table-body {
  margin: 0;
}

.h-modal-fullScreen .ant-modal-content .ant-modal-close-x {
  line-height: 56px !important;
}
</style>