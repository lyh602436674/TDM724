<!--
 * @Author: 雷宇航
 * @Date: 2022-06-22 14:39:51
 * @fileName: NewTestProjectFormItem.vue
 * @FilePath: tdm200-client\src\views\hifar\hifar-environmental-test\entrustment\components\NewTestProjectFormItem.vue
 * @Description: 对此文件的描述...
-->
<template>
  <div id="projectFormItem" style="height: 100%; position: relative">
    <h-form
      :ref="'projectInfoForm' + index"
      v-model="model"
      :column="3"
      :formData="formData"
      style="margin-top: 18px; margin-left: 15px"
    >
    </h-form>
    <h-collapse :activeKey="1" :forceRender="false" class="collapseStyle" title="试验条件">
      <!-- 项目信息中，填写试验条件功能移植到试验任务中由试验员填写，这里暂时用v-show隐藏掉（不能注释，不能使用v-if），其他js业务代码没有注释，以保证最小改动避免引发其他问题 -->
      <div slot="extraBox" v-if="filterUnitCode(model.unitCode)" v-show="false">
        <a-button size="small" type="primary" @click.stop="previewEcharts"> 预览</a-button>
      </div>
      <div slot="content" v-show="false">
        <template v-if="filterUnitCode(model.unitCode)">
          <a-tabs id="drag-tab" v-model="tabsActiveKey" hideAdd type="editable-card" @edit="onEdit">
            <template v-for="(item,itemIndex) in model.abilityRequire">
              <a-tab-pane :key="itemIndex" :closable="item.closable"
                          :tab="item.type === 'stage' ? item.title + itemIndex : item.title" forceRender>
                <test-condition-tab-item :key="itemIndex + '-only'" ref="testConditionTabItem"
                                         :itemAbilityInfo="item.abilityInfo"
                                         :itemIndex="itemIndex"
                                         :projectIndex="index"
                                         :currentProject="model"
                                         @add="handleAdd"
                                         @delete="handleDelete"/>
              </a-tab-pane>
            </template>
            <a-button slot="tabBarExtraContent" size="small" type="primary" @click="addLoopStage">新增循环阶段</a-button>
          </a-tabs>
        </template>
        <template v-else>
          <test-condition-tab-item ref="testConditionTabItem" :currentProject="model"
                                   :itemAbilityInfo="model.abilityRequire[0].abilityInfo"
                                   :projectIndex="index" @add="handleAdd" @delete="handleDelete"/>
        </template>
      </div>
      <div slot="content">
        <h-form
          :ref="'testConditionDataForm' + index"
          v-model="model"
          :column="3"
          :formData="testConditionData"
          style="margin-top: 18px; margin-left: 15px"
        >
        </h-form>
      </div>
    </h-collapse>
    <point-list ref="PointList" @change="pointSelectChange"></point-list>
    <new-test-experimental-curve-modal ref="ExperimentalCurveModal"
                                       @change="curveUrlChange"></new-test-experimental-curve-modal>
    <div id='echarts-result' style='display: none;width: 1000px;height: 500px;'></div>
  </div>
</template>

<script>
import moment from 'moment'
import {cloneDeep, isObject} from 'lodash'
import MethodSelectModal from '@/views/hifar/hifar-environmental-test/components/MethodSelectModal'
import WorkcenterSelectModal from '@/views/hifar/hifar-environmental-test/components/WorkcenterSelectModal'
import NewSampleListModal from './NewSampleListModal'
import PointList from './PointList'
import EditorPointModal from '@/views/components/EditorPointModal'
import {filterDictTextByCache} from '@comp/_util/JDictSelectUtil'
import NewTestExperimentalCurveModal from './NewTestExperimentalCurveModal'
import sortable from 'sortablejs'
import TestConditionTabItem from "@views/hifar/hifar-environmental-test/entrustment/modules/TestConditionTabItem";
import drawCurveMixin from '@/views/hifar/hifar-environmental-test/entrustment/drawCurveMixin'
import {randomUUID} from "@/utils/util";
import entrustmentMixins from '@/views/hifar/hifar-environmental-test/entrustment/components/entrustmentMixins'

export default {
  mixins: [drawCurveMixin, entrustmentMixins],
  components: {
    TestConditionTabItem,
    NewTestExperimentalCurveModal,
    MethodSelectModal,
    WorkcenterSelectModal,
    NewSampleListModal,
    PointList,
    EditorPointModal,
  },
  props: {
    project: {
      type: Object,
      default: () => {
      }
    },
    index: {
      type: [String, Number],
      default: null
    },
    drawerWidth: {
      type: Number
    },
    pieceTableData: {
      type: Array,
      default: () => []
    },
    entrustType: {
      type: [String, Number],
      default: ""
    }
  },
  watch: {
    project: {
      immediate: true,
      handler(val) {
        if (isObject(val) && Object.keys(val).length) {
          let obj = Object.assign({}, val)
          let filterUnitCodeFlag = this.filterUnitCode(obj.unitCode)
          obj.standardId = obj.standardId && obj.standardId != 0 ? obj.standardId : ''
          obj.unitId = obj.unitId ? obj.unitId : obj.id
          obj.testName = obj.unitName
          let fileList = []
          obj.fileInfo && obj.fileInfo.length && obj.fileInfo.forEach(item => {
            fileList.push({
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
              replaceStatus: item.replaceStatus
            })
          })
          obj.attachIds = fileList || []
          if (obj.abilityRequire && obj.abilityRequire.length) {
            obj.abilityRequire.forEach((item, index) => {
              item.closable = index !== 0 && index !== 1
            })
          } else {
            if (filterUnitCodeFlag) {
              obj.abilityRequire = [
                {
                  title: "前置处理", type: "before", closable: false, abilityInfo: [
                    {
                      paramId: randomUUID(),
                      paramName: "初始类型",
                      minValue: "1",
                      conditionTypeDesc: "1",
                      dataType: "select"
                    }
                  ]
                },
                {
                  title: "循环阶段", type: "stage", closable: false, abilityInfo: obj.abilityInfo.map(a => {
                    return {
                      paramId: a.abilityParamId,
                      ...a
                    }
                  })
                }
              ]
            } else {
              obj.abilityRequire = [
                {
                  title: "试验条件", type: "default", abilityInfo: obj.abilityInfo.map(a => {
                    return {
                      paramId: a.abilityParamId,
                      ...a
                    }
                  })
                }
              ]
            }
          }
          this.model = obj
          if (filterUnitCodeFlag) {
            this.$nextTick(() => {
              this.createSortable()
            })
          }
        }
      }
    },
    'model.pieceNos': {
      immediate: true,
      handler(val) {
        if (val) {
          this.$nextTick(() => {
            this.$refs.newSampleListModal.localSelectedName = val
          })
        }
      },
    }
  },
  data() {
    const validatorTestCondition = (rule, value, callback) => {
      if (value.length >= 240) {
        callback('最多输入240个字符，其他试验条件请以附件的形式提供！')
      }
      callback()
    }
    return {
      moment,
      tabsActiveKey: 0,
      model: {},
      selectRow: null,
      rowIndex: '',
      currentSumMark: '',
      testConditionData: [
        {
          title: '试验条件',
          key: 'testCondition',
          formType: 'textarea',
          span: 3,
          placeholder: '请输入试验条件',
          autoSize: {minRows: 4},
          maxLength: 240,
          rows: 4,
          validate: {
            rules: [{required: true, message: '请输入试验条件'}, {validator: validatorTestCondition}],
          },
        },
        {
          title: '附件',
          key: 'attachIds',
          span: 3,
          component: (
            <h-upload-img
              multiple={false}
              accept={"image/png,image/gif,image/jpg,image/jpeg"}
              v-decorator={['attachIds', {initialValue: []}]}
            />
          ),
        }
      ],
      formData: [
        {
          title: '',
          key: 'unitId',
          formType: 'input',
          hidden: true
        },
        {
          title: '',
          key: 'standardName',
          formType: 'input',
          hidden: true
        },
        {
          title: '',
          key: 'curveUrl',
          formType: 'input',
          hidden: true
        },
        {
          title: '名称',
          key: 'unitName',
          formType: 'text'
        },
        {
          title: '试验名称',
          key: 'testName',
          formType: 'input',
          validate: {
            rules: [{required: true, message: '请输入试验名称'}]
          }
        },
        {
          title: '试验标准',
          key: 'standardId',
          validate: {
            rules: [{required: true, message: '请选择试验标准'}]
          },
          component: (
            <method-select-modal
              v-decorator={['standardId', {initialValue: []}]}
              selectedName={() => {
                return this.model.standardName
              }}
              itemUnitId={() => {
                return this.model.unitId
              }}
              onchange={(selectedRowKeys, selectedRows) => {
                let formName = 'projectInfoForm' + this.index
                let standardName = selectedRows[0] ? selectedRows[0].standardCode + '-'+ selectedRows[0].standardName : ''
                this.model.standardId = selectedRowKeys
                this.model.standardName = standardName
                this.$refs[formName].form.setFieldsValue({standardName: standardName})
              }}
            />
          )
        },
        {
          title: '',
          key: 'pieceNos',
          formType: 'input',
          hidden: true
        },
        {
          title: '已选试品',
          key: 'pieceIds',
          validate: {
            rules: [{required: true, message: '请选择试品'}]
          },
          component: (
            <new-sample-list-modal
              type="checkbox"
              ref="newSampleListModal"
              v-decorator={['pieceIds', {initialValue: []}]}
              selectedName={() => {
                return this.model.pieceNosName ? this.model.pieceNosName : this.model.pieceNos
              }}
              disabled={+this.entrustType === 1}
              pieceTableData={this.pieceTableData}
              onchange={(selectedRowKeys, selectedRows, pieceNos) => {
                let formName = 'projectInfoForm' + this.index
                this.model.pieceIds = selectedRowKeys
                this.model.pieceNosName = pieceNos && pieceNos.length > 0 ? pieceNos.join(',') : ''
                this.$refs[formName].form.setFieldsValue({pieceNos: pieceNos.join(',')})
              }}
            />
          )
        },
        {
          title: '试验依据',
          key: 'checkRequire',
          formType: 'input'
        },
        {
          title: '总号',
          key: 'sumMark',
          formType: 'input',
          validate: {
            initialValue: '1',
            rules: [
              {required: true, message: '请输入总号'},
              {validator: this.validateSumMark}
            ]
          }

        },
        {
          title: '分号',
          key: 'separateMark',
          formType: 'input',
          validate: {
            initialValue: '1',
            rules: [
              {required: true, message: '请输入分号'},
              {validator: this.validateSeparateMark}
            ]
          }
        },
      ],
      eChartResult: {},
      curveUrl: '',
      selectBeforeProjectIndex: 0,
      selectBeforeItemIndex: 0,
      dragFlag: true
    }
  },
  mounted() {
  },
  methods: {
    createSortable() {
      try {
        let dragTab = document.getElementById('drag-tab').querySelector('.ant-tabs-nav').firstChild
        let dom = dragTab.querySelectorAll('.ant-tabs-tab');
        [].forEach.call(dom, ((item, index) => {
          item.classList.remove('disabledDrag')
          //给第一个dom添加 不可以进行拖动的类名
          if (index === 0) {
            item.classList.add('disabledDrag')
          }
        }))
        if (this.model.abilityRequire.length > 0) {
          this.model.abilityRequire.map((item, index) => {
            item.closable = index !== 0 && index !== 1;
          })
        }
        // this.$forceUpdate()
        this.$nextTick(() => {
          sortable.create(dragTab, {
            filter: ".disabledDrag",//不需要进行拖动的元素
            onEnd: this.dragTabEndFun,
            onMove: this.dragTabMoveFun,
          })
        })
      } catch {
      }
    },
    dragTabMoveFun(customEvent, originalEvent) {
      if (customEvent.related.className.includes('disabledDrag') && customEvent.related.innerText === '前置处理') {
        if (this.dragFlag) {
          this.$message.warning('循环阶段不能在前置处理前')
          this.dragFlag = false
        }
        return false
      }
      if (customEvent.related.className.includes('disabledDrag') && customEvent.related.innerText === '后置处理') {
        if (this.dragFlag) {
          this.$message.warning('循环阶段不能在后置处理后')
          this.dragFlag = false
        }
        return false
      }
    },
    dragTabEndFun(customEvent) {
      let {oldIndex, newIndex, to, from, item} = customEvent
      to.removeChild(item)
      from.insertBefore(item, to.children[oldIndex])
      let {abilityRequire} = this.model
      const currentTabItem = abilityRequire.splice(oldIndex, 1)[0]
      abilityRequire.splice(newIndex, 0, currentTabItem)
      abilityRequire.map((item, index) => {
        item.closable = index !== 0 && index !== 1;
      })
      setTimeout(() => {
        this.tabsActiveKey = newIndex
      }, 0)
    },
    onEdit(index, type) {
      if (type === 'remove') {
        this.model.abilityRequire.splice(index, 1)
        setTimeout(() => {
          this.tabsActiveKey = index - 1
        }, 0)
      }
    },
    curveUrlChange(url) {
      this.curveUrl = url;
      this.$refs['projectInfoForm' + this.index].form.setFieldsValue({
        curveUrl: url
      })
    },
    validateSumMark(rule, value, callback) {
      // 分号
      let separateMark = this.$refs['projectInfoForm' + this.index].form.getFieldValue('separateMark')
      if (!value) {
        callback('')
      } else {
        if (value < 0) {
          callback('请输入正确格式的分号')
        } else if (parseInt(separateMark) > parseInt(value)) {
          callback('总号不能小于分号')
        } else if (!(value.startsWith(this.currentSumMark))) {
          this.$refs['projectInfoForm' + this.index].form.setFieldsValue({sumMark: this.currentSumMark})
          callback()
        } else {
          callback()
        }
      }
    },
    validateSeparateMark(rule, value, callback) {
      // 总号
      let sumMark = this.$refs['projectInfoForm' + this.index].form.getFieldValue('sumMark')
      if (!value) {
        callback('')
      } else {
        if (value < 0) {
          callback('请输入正确格式的总号')
        } else if (parseInt(value) > parseInt(sumMark)) {
          callback('分号不能大于总号')
        } else {
          callback()
        }
      }
    },
    // 删除指标
    removeEvent(row, index) {
      const $table = this.$refs['pointTable' + [index]]
      $table.remove(row)
    },
    //预览
    previewEcharts() {
      this.splitByCurveType()
      let {temperatureResult, humidityResult, temperatureCurveFlag, humidityCurveFlag} = this
      let obj = {temperatureResult, humidityResult}
      // if (!temperatureCurveFlag) {
      //   return this.$message.warning('温度曲线绘制失败，请检查温度试验条件填写是否正确')
      // }
      // if (!humidityCurveFlag) {
      //   return this.$message.warning('湿度曲线绘制失败，请检查湿度试验条件填写是否正确')
      // }
      setTimeout(() => {
        this.$refs.ExperimentalCurveModal.open(obj)
      }, 0)
    },
    momentFormat(value) {
      let hours = moment.duration(value).get('hours')
      let minutes = moment.duration(value).get('minutes')
      return (hours < 10 ? ('0' + hours) : hours) + ':' + (minutes < 10 ? ('0' + minutes) : minutes)
    },
    resultEcharts() {
      this.splitByCurveType()
      let {temperatureCurveFlag, humidityCurveFlag} = this
      if (temperatureCurveFlag || humidityCurveFlag) {
        let chart = this.$echarts.init(document.getElementById('echarts-result'));
        let option = {
          title: [],//
          legend: {},
          animation: false,  // 去除动画
          xAxis: {
            name: "时长/ min",
            type: 'time',
            splitLine: {
              show: false
            },
            legend: {
              data: ['规划曲线']
            },
            splitNumber: 20,
            axisLine: {
              lineStyle: {
                color: '#1B2232'
              }
            },
            axisLabel: {
              formatter: function (value, index) {
                return this.momentFormat(value,['hours', 'minutes',]);
              }.bind(this),
              color: '#1B2232',
              rotate: 45,
            }
          },
          yAxis: {
            name: '数值',
            type: 'value',
            splitLine: {
              show: true
            },
            axisLine: {
              show: true,
              lineStyle: {
                color: '#1B2232'
              }
            },
            axisLabel: {
              color: '#1B2232'
            }
          },
          series: [
            {
              name: '温度/℃',
              type: 'line',
              hoverAnimation: false,
              symbolSize: 4,
              data: this.temperatureResult || []
            },
            {
              name: '湿度/RH',
              type: 'line',
              hoverAnimation: false,
              symbolSize: 4,
              data: this.humidityResult || []
            },
          ]
        }
        chart.setOption(option)
        this.curveUrl = chart.getDataURL()
      }
    },
    //新增循环阶段
    addLoopStage() {
      // let item = {title: "循环阶段", type: "stage", closable: true, abilityInfo: []}
      this.model.abilityRequire.push(Object.assign({},cloneDeep(this.model.abilityRequire[this.model.abilityRequire.length - 1]),{closable:true}))
      this.$nextTick(() => {
        this.createSortable()
        this.tabsActiveKey = this.model.abilityRequire.length - 1
      })
    },
    // 新增指标项
    handleAdd(projectIndex, itemIndex) {
      this.selectBeforeProjectIndex = projectIndex
      this.selectBeforeItemIndex = itemIndex
      let testConditionTab = this.$refs.testConditionTabItem
      let filterUnitCodeFlag = this.filterUnitCode(this.project.unitCode)
      let $projectTable
      if (filterUnitCodeFlag) {
        $projectTable = testConditionTab[itemIndex].$refs['pointTable' + [projectIndex] + [itemIndex]]
      } else {
        $projectTable = testConditionTab.$refs['pointTable' + [projectIndex] + [itemIndex]]
      }
      let pointInfoAllData = $projectTable.getData()
      let pointInfoDeleteData = $projectTable.getRecordset().removeRecords
      let pointInfoInsertData = $projectTable.getRecordset().insertRecords
      pointInfoAllData = pointInfoAllData.concat([], pointInfoInsertData)
      // 去掉已删除的数据
      for (let k of pointInfoDeleteData) {
        pointInfoAllData = pointInfoAllData.filter((item) => item !== k)
      }
      this.model.abilityRequire[itemIndex].abilityInfo = pointInfoAllData
      this.$refs.PointList.showSelectModal(this.model.abilityRequire[itemIndex])
    },
    handleDelete(row, rowIndex, projectIndex, itemIndex) {
      this.model.abilityRequire[itemIndex].abilityInfo.splice(rowIndex, 1)
    },
    // 指标项--新增change
    async pointSelectChange(selectedRowKeys, selectedRows) {
      selectedRows.forEach((item) => {
        this.model.abilityRequire[this.selectBeforeItemIndex].abilityInfo.push({
          paramId: item.id,
          paramName: item.paramName,
          paramType: item.paramType,
          paramTypeText: filterDictTextByCache('hf_dev_param_type', item.paramType) || '-',
          paramCode: item.paramCode,
          dataType: item.dataType,
          unitCode: item.unitCode,
          unitName: item.unitName,
          conditionTypeDesc: item.conditionTypeDesc,
          conditonType: item.conditonType ? item.conditonType : 'A01',
          maxValue: item.maxValue,
          minValue: item.minValue,
          standardValue: item.standardValue,
          strValue: item.strValue,
          curveType: item.curveType
        })
      })
    },
    splitByCurveType() {
      let testConditionTab = this.$refs.testConditionTabItem;
      let pointTableItem = [];
      [].forEach.call(testConditionTab, (item, i) => {
        pointTableItem.push(item.$refs['pointTable' + [this.index] + [i]].getData())
      })
      try {
        this.isHighTemperature = pointTableItem[0].filter(item => item.paramName === '初始类型')[0].conditionTypeDesc === '1'
      } catch {
        this.isHighTemperature = true
      }
      let drawTemperatureCurve = []
      let drawHumidityCurve = []
      let pointTableItemExtend = cloneDeep(pointTableItem).splice(1)
      for (let i = 0; i < pointTableItemExtend.length; i++) {
        let item = pointTableItemExtend[i]
        try {
          this.loopNum = item.filter(v => +v.curveType === 1 && v.paramCode === 'qh07')[0].conditionTypeDesc
        } catch {
          // return this.$message.warning(`循环阶段${i + 1}，请添加循环次数`)
          this.loopNum = 1
        }
        drawTemperatureCurve = drawTemperatureCurve.concat(this.drawTemperatureCurve(item.filter(v => +v.curveType === 1)))
        drawHumidityCurve = drawHumidityCurve.concat(this.drawHumidityCurve(item.filter(v => +v.curveType === 2)))
      }
      // let filterTemperature = pointTableItem.filter(item => +item.curveType === 1)
      // let filterHumidity = pointTableItem.filter(item => +item.curveType === 2)
      // try {
      //   this.loopNum = pointTableItem.filter(item => +item.curveType === 1 && item.paramCode === 'qh07')[0].conditionTypeDesc
      //   this.isHighTemperature = pointTableItem.filter(item => item.paramName === '初始类型')[0].conditionTypeDesc === '1'
      // } catch {
      //   this.isHighTemperature = true
      //   this.loopNum = 1
      // }

      // 温度
      // let drawTemperatureCurve = this.drawTemperatureCurve(filterTemperature)
      this.temperatureResult = drawTemperatureCurve.map(item => item.result).flat()
      this.temperatureCurveFlag = !drawTemperatureCurve.map(item => item.flag).some(item => item === false)
      // 湿度
      // let drawHumidityCurve = this.drawHumidityCurve(filterHumidity)
      this.humidityResult = drawHumidityCurve.map(item => item.result).flat()
      this.humidityCurveFlag = !drawHumidityCurve.map(item => item.flag).some(item => item === false)
      // 预计时长
      if (this.temperatureResult.length || this.humidityResult.length) {
        this.predictDuration = (Math.max(this.temperatureResult.length ? this.temperatureResult[this.temperatureResult.length - 1].name : 0, this.humidityResult.length ? this.humidityResult[this.humidityResult.length - 1].name : 0) / 1000 / 3600).toFixed(1)
        this.predictDuration = +this.predictDuration < 1 ? 1 : this.predictDuration
      } else {
        this.predictDuration = 0
      }
      this.initialCurveData()
    },
    initialCurveData() {
      this.initialTemTime = moment(0).format('x')//温度湿度初始时间
      this.initialHumTime = moment(0).format('x')//温度湿度初始时间
      this.initialTemperature = 25 // 初始温度
      this.initialHumidity = 30 // 初始湿度
    },
  }
}
</script>
<style lang="less" scoped>
.Exper-imental-curve {
  width: 100%;
  height: 100%;
}
</style>