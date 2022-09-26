<!--
 * @Author: 赵峰
 * @Date: 2021-11-12 15:09:59
 * @LastEditTime: 2021-11-30 15:37:30
 * @LastEditors: 马潭龙
 * @Descripttion: 试验信息--基本信息详情组件
 * @FilePath: \hifar-platform-client\src\views\hifar\hifar-environmental-test\task\components\TestBaseDetail.vue
-->
<template>
  <div style="height: 100%">
    <!-- 基本信息 -->
    <a-tabs v-model="activeTab" size="small" type="card" @change="handleTabsChange">
      <a-tab-pane v-for="(item, index) in entrustInfo" :key="index"
                  :tab="item.flag ?item.title : item.entrustCode "></a-tab-pane>
    </a-tabs>
    <template v-if="entrustInfo[activeTab]['flag']">
      <abnormal-record-table v-if="entrustInfo[activeTab]['type'] === 'exception'"
                             ref="AbnormalRecordTable"
                             :records="detailData" :style="{ marginTop: top ? top : '50px',height:'100%' }" isReadOnly/>
      <termination-record-table v-if="entrustInfo[activeTab]['type'] === 'end'"
                                ref="TerminationRecordTable"
                                :records="detailData" :style="{ marginTop: top ? top : '50px',height:'100%' }"/>
      <test-report-info v-if="entrustInfo[activeTab]['type'] === 'report'"
                        ref='TestReportInfo' :style="{ marginTop: top ? top : '50px',height:'100%' }"
                        :testId="detailData.id" class='autoHeight'/>
    </template>
    <template v-else>
      <!-- 基本信息 -->
      <h-desc id="basicInfo" ref="basicInfo" :style="{ marginTop: top ? top : '50px' }" lableWidth="110px"
              title="基本信息">
        <h-desc-item label="委托单号">{{ entrustInfoItem.entrustCode || '--' }}
          <a-button icon="eye" size="small" style="margin-left: 10px" type="primary" @click="entrustReview">委托单预览
          </a-button>
        </h-desc-item>
        <h-desc-item label="委托状态">
          <a-tag slot="content" :color="entrustInfoItem.status | wtStatusColorFilter" class="status_tag">
            {{ entrustInfoItem.status | wtStatusFilter }}
          </a-tag>
        </h-desc-item>
        <h-desc-item label="委托日期">
          {{
            entrustInfoItem.entrustTime && entrustInfoItem.entrustTime != 0
              ? moment(parseInt(entrustInfoItem.entrustTime)).format('YYYY-MM-DD')
              : '--'
          }}
        </h-desc-item>
        <h-desc-item label="委托单类型">{{ entrustInfoItem.entrustType | entrustTypeFilter }}</h-desc-item>
        <h-desc-item label="送试单位">{{ entrustInfoItem.custName || '--' }}</h-desc-item>
        <h-desc-item label="联系人">{{ entrustInfoItem.linkName || '--' }}</h-desc-item>
        <h-desc-item label="联系方式">{{ entrustInfoItem.linkMobile || '--' }}</h-desc-item>
        <h-desc-item label="客户地址" v-if="entrustInfoItem.entrustType == 2">{{ entrustInfoItem.custAddress || '--' }}</h-desc-item>
        <h-desc-item label="任务编码">{{ entrustInfoItem.outSourceCode || '--' }}</h-desc-item>
        <h-desc-item label="试验性质">{{ entrustInfoItem.testPropertyName || '--' }}</h-desc-item>
        <h-desc-item label="密级">{{ entrustInfoItem.secretLevelCode_dictText || '--' }}</h-desc-item>
        <h-desc-item label="是否出报告">
          {{ entrustInfoItem.isReport == 1 ? '是' : entrustInfoItem.isReport == 2 ? '否' : '--' }}
        </h-desc-item>
        <h-desc-item label='是否外包'>
          <span v-if="entrustInfoItem.isExternalManage == 1" slot="content" style="color:red">是</span>
          <span v-else slot="content" style="color:green">否</span>
        </h-desc-item>
        <h-desc-item label="创建人">
          {{ entrustInfoItem.createUserName ? entrustInfoItem.createUserName : '--' }}
        </h-desc-item>
        <h-desc-item label="创建时间">
          {{
            entrustInfoItem.createTime && entrustInfoItem.createTime != 0
              ? moment(parseInt(entrustInfoItem.createTime)).format('YYYY-MM-DD HH:mm:ss')
              : '--'
          }}
        </h-desc-item>
        <h-desc-item :span="3" label="委托单附件">
          <div slot="content">
            <template v-if="entrustInfoItem.attachInfo && entrustInfoItem.attachInfo.length">
              <div v-for="(item, index) in entrustInfoItem.attachInfo" :key="index" class="url-list">
                <span>{{ index + 1 }}、{{ item.fileName }}</span>
                <a-button
                  icon="download"
                  size="small"
                  type="primary"
                  @click="handleDownload(item.filePath, item.fileName)"
                >
                  下载
                </a-button>
              </div>
            </template>
            <span v-else>暂无附件</span>
          </div>
        </h-desc-item>
        <h-desc-item :span="3" label="备注">
          {{ entrustInfoItem.remarks ? entrustInfoItem.remarks : '--' }}
        </h-desc-item>
      </h-desc>
      <!--      试品信息-->
      <h-desc labelWidth="120px" title="试品信息">
        <a-table
          :columns="columns1[detailData.entrustInfo ? detailData.entrustInfo[0].entrustType : 1]"
          :dataSource="detailData.testPieceInfo"
          :pagination="false"
          bordered
          rowKey="id"
          size="small"
          style="width: 100%; height: 100%"
        ></a-table>
      </h-desc>

      <!-- 项目信息 -->
      <h-desc id="projectInfo" ref="projectInfo" lableWidth="110px" style="margin-top: 20px" title="项目信息">
        <h-desc-item label="项目名称">{{ projectInfo.unitName || '--' }}</h-desc-item>
        <h-desc-item label="试验名称">{{ projectInfo.testName || '--' }}</h-desc-item>
        <h-desc-item label="试验标准">{{ projectInfo.standardCode + projectInfo.standardName || '--' }}</h-desc-item>
        <h-desc-item label="预计时长(h)">{{ projectInfo.predictDuration || '--' }}</h-desc-item>
        <h-desc-item label="总号">
          {{ projectInfo.sumMark ? projectInfo.sumMark : '--' }}
        </h-desc-item>
        <h-desc-item label="分号">
          {{ projectInfo.separateMark ? projectInfo.separateMark : '--' }}
        </h-desc-item>
        <h-desc-item :span="3" label="试验依据">{{ projectInfo.checkRequire || '--' }}</h-desc-item>
        <h-desc-item :span="3" label="其他条件">{{ projectInfo.testCondition || '--' }}</h-desc-item>
        <h-desc-item :span='4' label='附件'>
          <div slot='content'>
            <template v-if='projectInfo.fileInfo && projectInfo.fileInfo.length'>
              <div v-for='(item, index) in projectInfo.fileInfo' :key='index' class='url-list'>
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
        <h-desc-item :span="3" label="试验条件要求">
          <template slot="content">
            <template v-if="!projectAbilityInfo.length">
              <a-empty style="width: 100%"></a-empty>
            </template>
            <a-tabs v-else :default-active-key="0" style="width: 100%">
              <template v-for="(proItem,itemIndex) in projectAbilityInfo">
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
                    style="width: 100%"
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
          </template>
        </h-desc-item>
        <h-desc-item label="规划曲线">
          <template slot="content">
            <img v-if="curveUrl" :src="curveUrl" alt="规划曲线" style="width: 100%;height:500px">
            <span v-else>暂无规划曲线</span>
          </template>
        </h-desc-item>
      </h-desc>
      <!-- 试验信息 -->
      <h-desc id="testInfo" ref="testInfo" lableWidth="110px" style="margin-top: 20px; margin-bottom: 20px"
              title="试验信息">
        <h-desc-item label="试验设备">{{ projectInfo.unitName || '--' }}</h-desc-item>
        <h-desc-item label="费用（元）">{{ detailData.testCost || '--' }}</h-desc-item>
        <h-desc-item label="负责人">{{ detailData.chargeUserName || '--' }}</h-desc-item>
        <h-desc-item label="开始时间">{{
            detailData.realStartTime && detailData.realStartTime != 0
              ? moment(parseInt(detailData.realStartTime)).format('YYYY-MM-DD HH:mm:ss')
              : '--'
          }}
        </h-desc-item>
        <h-desc-item label="结束时间">{{
            detailData.realEndTime && detailData.realEndTime != 0
              ? moment(parseInt(detailData.realEndTime)).format('YYYY-MM-DD HH:mm:ss')
              : '--'
          }}
        </h-desc-item>
        <h-desc-item label="试验部门">{{ projectInfo.workName || '--' }}</h-desc-item>
        <h-desc-item :span="3" label="参试人员">
          {{ testPersonInfo.length > 0 ? testPersonInfo.join(',') : '--' }}
        </h-desc-item>
        <h-desc-item :span="3" label="测试设备">
          {{ testEquipInfo.length > 0 ? testEquipInfo.join(',') : '--' }}
        </h-desc-item>
        <h-desc-item :span="3" label="试验结果">{{ detailData.processDesc || '--' }}</h-desc-item>
      </h-desc>
      <!-- 试前检查 -->
      <h-desc id="testBeforCheck" ref="testBeforCheck" :bordered='false' lableWidth="110px"
              style="margin-top: 20px; margin-bottom: 20px" title="试前检查">
        <h-vex-table
          ref="beforeCheckInfo"
          :columns="columns"
          :data="beforeCheckInfo"
          :pagination="false"
          bordered
          style="width: 100%; height: 200px"
        >
        <span slot="itemRes" slot-scope="text, record">
          <h-icon v-if="record.itemRes == '1'" class="danger-text" type="icon-chacha"/>
          <h-icon v-else-if="record.itemRes == '2'" class="success-text" type="icon-wancheng1"/>
        </span>
        </h-vex-table>
      </h-desc>
      <!-- 试中检查 -->
      <h-desc id="testInCheck" ref="testInCheck" :bordered='false' lableWidth="110px"
              style="margin-top: 20px; margin-bottom: 20px" title="试中检查">
        <h-vex-table
          ref="inCheckInfo"
          :columns="columns"
          :data="inCheckInfo"
          :pagination="false"
          bordered
          style="width: 100%; height: 200px"
        >
        <span slot="itemRes" slot-scope="text, record">
          <h-icon v-if="record.itemRes == '1'" class="danger-text" type="icon-chacha"/>
          <h-icon v-else-if="record.itemRes == '2'" class="success-text" type="icon-wancheng1"/>
        </span>
        </h-vex-table>
      </h-desc>
      <!-- 试后检查 -->
      <h-desc id="testAfterCheck" ref="testAfterCheck" :bordered='false' lableWidth="110px"
              style="margin-top: 20px; margin-bottom: 20px" title="试后检查">
        <h-vex-table
          ref="afterCheckInfo"
          :columns="columns"
          :data="afterCheckInfo"
          :pagination="false"
          bordered
          style="width: 100%; height: 200px"
        >
        <span slot="itemRes" slot-scope="text, record">
          <h-icon v-if="record.itemRes == '1'" class="danger-text" type="icon-chacha"/>
          <h-icon v-else-if="record.itemRes == '2'" class="success-text" type="icon-wancheng1"/>
        </span>
        </h-vex-table>
      </h-desc>
      <!-- 试验数据 -->
      <h-desc id="testData" ref="testData" title="试验数据">
        <div style="height: 100%; width: 100%; overflow: auto; padding: 20px">
          <h-desc id="imageForm" :bordered="false">
            <h-form ref="imageForm" v-model="model_img" :column="1" :formData="imageData" style="width: 100%"/>
          </h-desc>
<!--          <h-desc id="curveForm" :bordered="false">
            <h-form ref="curveForm" v-model="model_cure" :column="1" :formData="curveData" style="width: 100%"/>
          </h-desc>
          <h-desc id="atlasForm" :bordered="false">
            <h-form ref="atlasForm" v-model="model_atlas" :column="1" :formData="atlasData" style="width: 100%"/>
          </h-desc>-->
          <h-desc id="attachForm" :bordered="false">
            <h-form ref="attachForm" v-model="model_attach" :column="1" :formData="attachData"
                    style="width: 100%"/>
          </h-desc>
          <h-desc id="videoForm" :bordered="false">
            <h-form ref="videoForm" v-model="model_video" :column="1" :formData="videoData" style="width: 100%"/>
          </h-desc>
        </div>
      </h-desc>
    </template>
    <test-entrust-review-pdf ref="testEntrustReviewPdf"/>
  </div>
</template>

<script>
import moment from 'moment'
import {downloadFile, getFileAccessHttpUrl, postAction} from '@api/manage'
import mixin from '@/views/hifar/mixin.js'
import AbnormalRecordTable
  from '@/views/hifar/hifar-environmental-test/task/modules/components/detail/AbnormalRecordTable'
import TerminationRecordTable
  from '@/views/hifar/hifar-environmental-test/task/modules/components/detail/TerminationRecordTable'
import TestReportInfo from '@views/hifar/hifar-environmental-test/task/components/TestReportInfo'
import TestEntrustReviewPdf from "@views/hifar/hifar-environmental-test/task/modules/TestEntrustReviewPdf";

export default {
  components: {TestEntrustReviewPdf, AbnormalRecordTable, TerminationRecordTable, TestReportInfo},
  mixins: [mixin],
  props: {
    testId: {
      type: String,
      default: '',
    },
    top: {
      type: String,
      default: '',
    },
    showExceptionAndEnd: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      moment,
      activeTab: 0,
      checkId: '',
      curveUrl: '',
      url: {
        detail: '/HfEnvTaskTestBusiness/queryById',
        CheckInfo: '/HfEnvTaskTestBusiness/queryTestCheckItem',
        // 试验数据
        attachList: '/MinioBusiness/listByRefId',
        delete: '/MinioBusiness/logicRemoveById',
      },
      detailData: {},
      projectInfo: {},
      testEquipInfo: [],
      testPersonInfo: [],
      projectAbilityInfo: [],
      entrustInfo: [{flag: false}],
      entrustInfoItem: {},
      title: '',
      visible: false,
      model_cure: {},
      model_img: {},
      model_atlas: {},
      model_attach: {},
      model_video: {},
      imglength: 0,
      // testId: '',

      columns: [
        {
          title: '检查项名称',
          align: 'left',
          dataIndex: 'itemName',
          minWidth: 10,
          customRender: (text, record) => {
            return text || '--'
          },
        },
        {
          title: '检查项内容',
          align: 'left',
          dataIndex: 'itemContent',
          minWidth: 10,
          customRender: (text, record) => {
            return text || '--'
          },
        },
        {
          title: '检查项要求',
          align: 'left',
          dataIndex: 'itemRequire',
          minWidth: 25,
          customRender: (text, record) => {
            return text || '--'
          },
        },
        {
          title: '检查结果',
          align: 'left',
          dataIndex: 'itemRes',
          minWidth: 10,
          scopedSlots: {customRender: 'itemRes'},
        },
        {
          title: '检查人',
          align: 'left',
          dataIndex: 'fillUserName',
          customRender: (text, record) => {
            return text || '--'
          },
        },
        {
          title: '复核人',
          align: 'left',
          dataIndex: 'checkUserName',
          customRender: (text, record) => {
            return text || '--'
          },
        },
      ],
      // columns1: [
      //   {
      //     title: '试品名称',
      //     dataIndex: 'productName',
      //   },
      //   {
      //     title: '试品编号',
      //     dataIndex: 'productCode',
      //   },
      //   {
      //     title: '试品代号',
      //     dataIndex: 'productAlias',
      //   },
      //   {
      //     title: '试品编号',
      //     dataIndex: 'pieceNo'
      //   }
      // ],
      columns1: {
        1: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: 'center',
            customRender: function (t, r, index) {
              return parseInt(index) + 1
            },
          },
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
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: 'center',
            customRender: function (t, r, index) {
              return parseInt(index) + 1
            },
          },
          {
            title: '试品名称',
            dataIndex: 'productName',
          },
          {
            title: '试品型号',
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
      // 试前检查
      beforeCheckInfo: () => {
        return postAction(this.url.CheckInfo, {id: this.checkId}).then((res) => {
          if (res.code === 200) {
            return res.data.beforeCheckInfo
          }
        })
      },
      // 试中检查
      inCheckInfo: () => {
        return postAction(this.url.CheckInfo, {id: this.checkId}).then((res) => {
          if (res.code === 200) {
            return res.data.inCheckInfo
          }
        })
      },
      // 试后检查
      afterCheckInfo: () => {
        return postAction(this.url.CheckInfo, {id: this.checkId}).then((res) => {
          if (res.code === 200) {
            return res.data.afterCheckInfo
          }
        })
      },
      // 试验数据
      loadData() {
        this.loadImgData()
        this.loadCureData()
        this.loadAtlasData()
        this.loadAttachData()
        this.loadVideoData()
      },
      // 图片
      imageData: [
        {
          title: '曲线/图谱图片',
          key: 'attachIds',
          span: 1,
          component: (
            <h-upload-img
              isEdit={false}
              multiple={true}
              max={100}
              customParams={{refType: 'test_picture', refId: this.checkId}}
              accept="image/png,image/gif,image/jpg,image/jpeg"
              v-decorator={['attachIds', {initialValue: []}]}
              on-delete={this.handleDelete}
            />
          ),
        },
      ],
      // 曲线
      curveData: [
        {
          title: '曲线',
          key: 'attachIds',
          span: 1,
          component: (
            <h-upload-file
              isEdit={false}
              v-decorator={['attachIds', {initialValue: []}]}
              customParams={{refType: 'test_cure', refId: this.checkId}}
              on-delete={this.handleDelete}
            />
          ),
        },
      ],
      // 图谱
      atlasData: [
        {
          title: '图谱',
          key: 'attachIds',
          span: 1,
          component: (
            <h-upload-file
              isEdit={false}
              v-decorator={['attachIds', {initialValue: []}]}
              customParams={{refType: 'test_atlas', refId: this.checkId}}
              on-delete={this.handleDelete}
            />
          ),
        },
      ],
      // 附件
      attachData: [
        {
          title: '附件',
          key: 'attachIds',
          span: 1,
          component: (
            <h-upload-file
              isEdit={false}
              v-decorator={['attachIds', {initialValue: []}]}
              customParams={{refType: 'test_attach', refId: this.checkId}}
              on-delete={this.handleDelete}
            />
          ),
        },
      ],
      // 视频
      videoData: [
        {
          title: '视频',
          key: 'attachIds',
          span: 1,
          component: (
            <h-upload-file
              isEdit={false}
              v-decorator={['attachIds', {initialValue: []}]}
              customParams={{refType: 'test_video', refId: this.checkId}}
              on-delete={this.handleDelete}
            />
          ),
        },
      ],
    }
  },
  created() {
    this.loadData()
  },
  watch: {
    testId: {
      immediate: true,
      handler(val) {
        if (val) {
          this.loadDetailData(val)
        }
      },
    },
    top: {
      immediate: true,
      handler(val) {
        if (val) {
          this.top = val
        }
      },
    },
  },
  methods: {
    entrustReview() {
      this.$refs.testEntrustReviewPdf.show(this.entrustInfoItem.reportPath)
    },
    handleDownload(filePath, fileName) {
      let fileAccessUrl = getFileAccessHttpUrl(filePath)
      downloadFile(fileAccessUrl, fileName)
    },
    handleTabsChange(v) {
      this.activeTab = v
      this.entrustInfoItem = {}
      this.entrustInfoItem = this.entrustInfo[v]
    },
    loadDetailData(id) {
      this.checkId = id
      postAction(this.url.detail, {id: id}).then((res) => {
        if (res.code == 200) {
          const {data} = res
          let testEquipInfoArr = data.testEquipInfo
          let testPersonInfoArr = data.testPersonInfo
          let entrustInfoArr = data.entrustInfo
          let testEquipInfo = []
          let testPersonInfo = []
          this.detailData = data
          this.projectInfo = data.testTaskInfo[0]
          this.projectAbilityInfo = data.projectInfo[0].abilityRequire || []
          this.curveUrl = data.projectInfo[0].curveUrl || ''
          if (testEquipInfoArr.length) {
            testEquipInfoArr.forEach((item) => {
              testEquipInfo.push(item.equipName + (item.innerName ? '(' + item.innerName + ')' : ''))
            })
          }
          if (testPersonInfoArr.length) {
            testPersonInfoArr.forEach((item) => {
              testPersonInfo.push(item.testUserName + '(' + item.testPostName + ')')
            })
          }
          this.testEquipInfo = testEquipInfo
          this.testPersonInfo = testPersonInfo
          this.entrustInfoItem = entrustInfoArr[0]
          let entrustInfo = entrustInfoArr
          if (this.showExceptionAndEnd) {
            entrustInfo.push(
              {title: "报告信息", flag: true, type: "report"},
              {title: "异常记录", flag: true, type: "exception"},
              {title: "终止记录", flag: true, type: "end"},
            )
          }
          this.entrustInfo = entrustInfo
        }
      })
    },
    // 试验数据
    // 图片
    loadImgData() {
      postAction(this.url.attachList, {refType: 'test_picture', refId: this.checkId}).then((res) => {
        if (res.code == 200) {
          const {data} = res
          let fileArr = []
          let obj = {}
          if (data && data.length > 0) {
            data.forEach((item) => {
              fileArr.push({
                fileId: item.id,
                size: item.fileSize,
                status: item.status == 9 ? 'success' : 'exception',
                url: item.filePath,
                name: item.fileName,
                uuid: item.id,
                percent: 100,
                uploadTime: item.createTime,
                secretLevel: item.secretLevel,
                type: item.viewType == 2 ? 'image/jpeg' : 'text/plain',
              })
            })
          }
          obj.attachIds = fileArr
          this.model_img = obj
        }
      })
    },
    // 曲线
    loadCureData() {
      postAction(this.url.attachList, {refType: 'test_cure', refId: this.checkId}).then((res) => {
        if (res.code == 200) {
          const {data} = res
          let fileArr = []
          let obj = {}
          if (data && data.length > 0) {
            data.forEach((item) => {
              fileArr.push({
                fileId: item.id,
                size: item.fileSize,
                status: item.status == 9 ? 'success' : 'exception',
                url: item.filePath,
                name: item.fileName,
                uuid: item.id,
                percent: 100,
                uploadTime: item.createTime,
                secretLevel: item.secretLevel,
                type: item.viewType == 2 ? 'image/jpeg' : 'text/plain',
              })
            })
          }
          obj.attachIds = fileArr
          this.model_cure = obj
        }
      })
    },
    // 图谱
    loadAtlasData() {
      postAction(this.url.attachList, {refType: 'test_atlas', refId: this.checkId}).then((res) => {
        if (res.code == 200) {
          const {data} = res
          let fileArr = []
          let obj = {}
          if (data && data.length > 0) {
            data.forEach((item) => {
              fileArr.push({
                fileId: item.id,
                size: item.fileSize,
                status: item.status == 9 ? 'success' : 'exception',
                url: item.filePath,
                name: item.fileName,
                uuid: item.id,
                percent: 100,
                uploadTime: item.createTime,
                secretLevel: item.secretLevel,
                type: item.viewType == 2 ? 'image/jpeg' : 'text/plain',
              })
            })
          }
          obj.attachIds = fileArr
          this.model_atlas = obj
        }
      })
    },
    // 附件
    loadAttachData() {
      postAction(this.url.attachList, {refType: 'test_attach', refId: this.checkId}).then((res) => {
        if (res.code == 200) {
          const {data} = res
          let fileArr = []
          let obj = {}
          if (data && data.length > 0) {
            data.forEach((item) => {
              fileArr.push({
                fileId: item.id,
                size: item.fileSize,
                status: item.status == 9 ? 'success' : 'exception',
                url: item.filePath,
                name: item.fileName,
                uuid: item.id,
                percent: 100,
                uploadTime: item.createTime,
                secretLevel: item.secretLevel,
                type: item.viewType == 2 ? 'image/jpeg' : 'text/plain',
              })
            })
          }
          obj.attachIds = fileArr
          this.model_attach = obj
        }
      })
    },
    // 视频
    loadVideoData() {
      postAction(this.url.attachList, {refType: 'test_video', refId: this.checkId}).then((res) => {
        if (res.code == 200) {
          const {data} = res
          let fileArr = []
          let obj = {}
          if (data && data.length > 0) {
            data.forEach((item) => {
              fileArr.push({
                fileId: item.id,
                size: item.fileSize,
                status: item.status == 9 ? 'success' : 'exception',
                url: item.filePath,
                name: item.fileName,
                uuid: item.id,
                percent: 100,
                uploadTime: item.createTime,
                secretLevel: item.secretLevel,
                type: item.viewType == 2 ? 'image/jpeg' : 'text/plain',
              })
            })
          }
          obj.attachIds = fileArr
          this.model_video = obj
        }
      })
    },
    // 图片删除
    handleDelete(file, fileList) {
      postAction(this.url.delete, {id: file.fileId}).then(() => {
      })
    },
  },
}
</script>
<style lang='less' scoped>
/deep/ .ant-tabs {
  position: initial !important;
}

///deep/ .ant-tabs-bar {
//  margin: 0;
//}

.url-list {
  display: flex;
  align-items: center;
  margin-bottom: 10px;

  span {
    margin-right: 10px;
  }
}
</style>