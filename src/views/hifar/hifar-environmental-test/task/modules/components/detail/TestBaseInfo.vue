<!--
 * @Author: 赵峰
 * @Date: 2021-11-04 16:46:50
 * @LastEditTime: 2021-11-10 14:25:17
 * @LastEditors: 赵峰
 * @Descripttion: 试验设备基本信息
 * @FilePath: \hifar-platform-client\src\views\hifar\hifar-environmental-test\task\modules\components\detail\TestBaseInfo.vue
-->
<template>
  <div>
    <h-card fixed title="基本信息" style="height: auto" class="basicInfo" :showCollapse="true">
      <h-desc slot="content" size="small" labelWidth="120px">
        <h-desc-item
          v-for="(item, index) in basicData"
          :span="item.span ? item.span : 1"
          :key="index"
          :label="item.title"
        >
          {{
            item.isTime
              ? parseFloat(item.value) == 0
                ? '--'
                : moment(parseFloat(item.value)).format('YYYY-MM-DD HH:mm:ss')
              : item.value
              ? item.value
              : '--'
          }}
        </h-desc-item>
      </h-desc>
    </h-card>
    <h-card title="项目信息" v-for="(item, index) in projectData" :key="index" style="margin-bottom: 10px">
      <h-desc slot="content" size="small" labelWidth="130px">
        <h-desc-item label="项目名称">
          {{ item.unitName ? item.unitName : '--' }}
        </h-desc-item>
        <h-desc-item label="试验名称">
          {{ item.testName ? item.testName : '--' }}
        </h-desc-item>
        <h-desc-item label="试验标准">
          {{item.standardCode + item.standardName || '--' }}
        </h-desc-item>
        <h-desc-item label="预计时长（h）">
          {{ item.predictDuration ? item.predictDuration + '小时' : '--' }}
        </h-desc-item>
        <h-desc-item label="总号">
          {{ item.sumMark ? item.sumMark : '--' }}
        </h-desc-item>
        <h-desc-item label="分号">
          {{ item.separateMark ? item.separateMark : '--' }}
        </h-desc-item>
        <h-desc-item label="试验依据" :span="4">
          {{ item.checkRequire ? item.checkRequire : '--' }}
        </h-desc-item>
        <h-desc-item label="其他条件" :span="4">
          {{ item.testCondition ? item.testCondition : '--' }}
        </h-desc-item>
        <h-desc-item :span='4' label='附件'>
          <div slot='content'>
            <template v-if='item.fileInfo && item.fileInfo.length'>
              <div v-for='(item, index) in item.fileInfo' :key='index' class='url-list'>
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
            <template v-if="!item.abilityRequire.length"><a-empty style="width: 100%"></a-empty></template>
            <a-tabs v-else :default-active-key="0" style="width: 100%">
              <template v-for="(proItem,itemIndex) in item.abilityRequire">
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
      </h-desc>
    </h-card>
    <h-card title="试品信息" style="height: auto" :showCollapse="true">
      <div slot="content">
        <a-table
          size="small"
          :columns="testProductColumns[testTaskData[0].entrustType]"
          :dataSource="productTable"
          :pagination="false"
          bordered
          rowKey="id"
        >
        </a-table>
      </div>
    </h-card>
    <h-card title="参试人员" style="height: auto" :showCollapse="true">
      <div slot="content">
        <a-table
          ref="personTable"
          size="small"
          :columns="personColumns"
          :dataSource="personArr"
          :pagination="false"
          bordered
        >
        </a-table>
      </div>
    </h-card>
    <h-card title="测试设备" :style="{ marginBottom: 0, height: 'auto' }" :showCollapse="true">
      <div slot="content">
        <a-table size="small" :columns="testEquipColumns" :dataSource="equipData" :pagination="false" bordered>
        </a-table>
      </div>
    </h-card>
    <h-card title="传感器" :style="{ marginBottom: 0, height: 'auto' }" :showCollapse="true">
      <div slot="content">
        <a-table size="small" :columns="sensorColumns" :dataSource="sensorData" :pagination="false" bordered>
        </a-table>
      </div>
    </h-card>
    <h-card title="振动工装" :style="{ marginBottom: 0, height: 'auto' }" :showCollapse="true">
      <div slot="content">
        <a-table size="small" :columns="toolsProductColumns" :dataSource="toolsProductData" :pagination="false" bordered>
        </a-table>
      </div>
    </h-card>
  </div>
</template>

<script>
import moment from 'moment'
export default {
  props: {
    basicData: {
      type: [Array, Object],
      default: () => {},
    },
    testTaskData: {
      type: [Array, Object],
      default: () => {},
    },
    projectData: {
      type: [Array, Object],
      default: () => {},
    },
    productTable: {
      type: [Array, Object],
      default: () => {},
    },
    carryOutProcessData: {
      type: [Array, Object],
      default: () => {},
    },
    personArr: {
      type: [Array, Object],
      default: () => {},
    },
    equipData: {
      type: [Array, Object],
      default: () => {},
    },
    sensorData: {
      type: [Array, Object],
      default: () => {},
    },
    toolsProductData: {
      type: [Array, Object],
      default: () => {},
    },
  },
  components: {},
  data() {
    return {
      moment,
      toolsProductColumns: [
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
          title: '工装编号',
          dataIndex: 'toolsCode',
          customRender: (t) => {
            return t ? t : '--'
          }
        },
        {
          title: '工装名称',
          dataIndex: 'toolsName',
          customRender: (t) => {
            return t ? t : '--'
          }
        },
        {
          title: '工装规格',
          dataIndex: 'toolsSize',
          customRender: (t) => {
            return t ? t : '--'
          }
        },
        {
          title: '在库状态',
          dataIndex: 'larbaryStatus_dictText',
          customRender: (text) => {
            return text || '--'
          }
        },
        {
          title: '存放地点',
          dataIndex: 'location',
          customRender: (t) => {
            return t ? t : '--'
          }
        },
        {
          title: '责任部门',
          dataIndex: 'deptName',
          customRender: (t) => {
            return t ? t : '--'
          }
        },
        {
          title: '设备状态',
          dataIndex: 'toolsStatus_dictText',
          align: 'left',
          width: 120,
          customRender: (text, record) => {
            return text || '--'
          }
        },
        {
          title: '工装分类',
          dataIndex: 'classify_dictText',
          customRender: (text) => {
            return text || '--'
          }
        },
      ],
      sensorColumns: [
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
          title: '设备编号',
          dataIndex: 'equipCode',
          customRender: (t) => {
            return t ? t : '--'
          }
        },
        {
          title: '设备名称',
          dataIndex: 'equipName',
          customRender: (t) => {
            return t ? t : '--'
          }
        },
        {
          title: '计量有效期',
          dataIndex: 'checkTime',
          customRender: (t, record) => {
            return +record.checkTime && moment(+record.checkTime).format('YYYY-MM-DD') || '--'
          }
        },
        {
          title: '设备型号',
          dataIndex: 'equipModel',
          customRender: (t) => {
            return t ? t : '--'
          }
        },
      ],
      testTaskColumns: [
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
          title: '试验名称',
          dataIndex: 'testName',
          customRender: (text) => {
            return text ? text : '--'
          },
        },
        {
          title: '试验中心',
          dataIndex: 'workName',
          customRender: (text) => {
            return text ? text : '--'
          },
        },
        {
          title: '试验条件',
          dataIndex: 'testCondition',
          customRender: (text) => {
            return text ? text : '--'
          },
          width: 180,
        },
        {
          title: '委托单号',
          dataIndex: 'entrustCode',
          customRender: (text) => {
            return text ? text : '--'
          },
        },
        {
          title: '委托单位',
          dataIndex: 'custName',
          customRender: (text) => {
            return text ? text : '--'
          },
        },
        {
          title: '任务编号',
          dataIndex: 'taskCode',
          customRender: (text) => {
            return text ? text : '--'
          },
        },
        {
          title: '项目名称',
          dataIndex: 'unitName',
          customRender: (text) => {
            return text ? text : '--'
          },
        },
        {
          title: '检验要求',
          dataIndex: 'checkRequire',
          customRender: (text) => {
            return text ? text : '--'
          },
        },
      ],
      projectColumns: [
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
          title: '委托单号',
          dataIndex: 'entrustCode',
          customRender: (t) => {
            return t ? t : '--'
          },
        },
        {
          title: '试验编号',
          dataIndex: 'testCode',
          customRender: (t) => {
            return t ? t : '--'
          },
        },
        {
          title: '委托单类型',
          dataIndex: 'entrustType',
          customRender: (t) => {
            return t ? t : '--'
          },
        },
        {
          title: '试验性质',
          dataIndex: 'testPropertyName',
          customRender: (t) => {
            return t ? t : '--'
          },
        },
        {
          title: '项目编码',
          dataIndex: 'unitCode',
          customRender: (t) => {
            return t ? t : '--'
          },
        },
        {
          title: '项目名称',
          dataIndex: 'unitName',
          customRender: (t) => {
            return t ? t : '--'
          },
        },
      ],
      testProductColumns:{
        1:[
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
        2:[
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
      personColumns: [
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
        { title: '参试人员岗位', dataIndex: 'testPostName' },
        { title: '参试人员姓名', dataIndex: 'testUserName' },
      ],
      testEquipColumns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function (t, r, index) {
            return parseInt(index) + 1
          }
        },
        {title: '设备编号', dataIndex: 'equipCode'},
        {title: '设备名称', dataIndex: 'equipName'},
        {
          title: '计量有效期',
          dataIndex: 'checkValid',
          customRender: (t, record) => {
            return +record.checkValid && moment(+record.checkValid).format('YYYY-MM-DD') || '--'
          }
        },
        {title: '设备型号', dataIndex: 'equipModel'},
      ],
      sensorData: [],
      toolsProductData: [],
    }
  },
  methods: {},
}
</script>
<style lang='less' scoped>
</style>