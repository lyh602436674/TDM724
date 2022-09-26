<!--
 * @Author: 马潭龙
 * @Date: 2021-11-15 11:24:21
 * @LastEditTime: 2021-11-26 18:25:57
 * @LastEditors: 马潭龙
 * @Descripttion: 设备详情顶部-选项卡-曲线图
 * @FilePath: \hifar-platform-client\src\views\hifar\hifar-environmental-test\task\modules\components\equipBasicEcharts\EquipBasicLine.vue
-->
<template>
  <div style="height: 100%">
    <div style="height: 13.5%;margin-left: 5px;display: flex">
      <div style="margin-right: 10px">
        <h-range-picker
          v-model="queryParams.initialTimeRange"
          :placeholder="['开始时间', '结束时间']"
          format="YYYY-MM-DD HH:mm:ss"
          show-time
          size="small"
          style="width: 100%"
          @change="onDateChange"
        >
          <span slot="addonBefore">时间选择</span>
        </h-range-picker>
      </div>
      <div style="margin-right: 10px;flex: 1;display: flex">
        <a-input style="margin-right: 10px;" v-model="queryParams.initialTemperature" placeholder="请输入初始温度" size="small"
                 type="number">
          <span slot="addonBefore">初始温度</span>
        </a-input>
        <a-input v-model="queryParams.initialHumidity" placeholder="请输入初始湿度"
                 size="small" type="number">
          <span slot="addonBefore">初始湿度</span>
        </a-input>
      </div>
      <div style="margin-right: 10px;flex: 1">
        <h-date v-model="queryParams.initialTime" :show-time="true" format="YYYY-MM-DD HH:mm:ss" placeholder="请输入初始时间"
                size="small">
          <span slot="addonBefore">初始时间</span>
        </h-date>
      </div>
      <div style="flex: 1">
        <a-button icon="search" size="small" type="primary" @click="handleSearch">查询</a-button>
        <a-button icon="reload" size="small" style="margin: 0 0 0 5px" type="default" @click="handleReset">
          重置
        </a-button>
        <a-button v-if="enlargementShow" icon="fullscreen" size="small" style="margin: 0 0 0 5px" type="primary"
                  @click="enlargement">
          放大
        </a-button>
      </div>
    </div>
    <div v-if="spinLoading" class="spin-box">
      <a-spin/>
    </div>
    <div v-else id="eCharts" :style="{ width: '100%', height: '86.5%' }"></div>
  </div>
</template>

<script>
import {getAction, postAction} from '@api/manage'
import moment from 'moment'
import drawCurveMixin from "@views/hifar/hifar-environmental-test/entrustment/drawCurveMixin";

export default {
  props: {
    equipCode: {
      type: String,
      default: ''
    },
    selectedRow: {
      type: Array,
      default: []
    }
  },
  mixins: [drawCurveMixin],
  watch: {
    equipCode: {
      immediate: true,
      handler(val) {
        this.equipCodeProps = val
      }
    },
    selectedRow: {
      immediate: true,
      deep: true,
      handler(row) {
        if (row && row.length) {
          this.leftTreeChange = false
          this.projectIds = row[0].projectIds
          this.initialTemTime = moment(this.queryParams.initialTime).format('x')
          this.initialHumTime = moment(this.queryParams.initialTime).format('x')
          if (this.enlargementShow && !this.leftTreeChange) {
            this.handleSearch()
          }
        }
      }
    },
    selectedTreeRows: {
      immediate: true,
      deep: true,
      handler(row) {
        this.leftTreeChange = true
      }
    },
  },
  data() {
    return {
      url: {
        listAll: '/DaqBusiness/listAll',
        queryByProjectId: 'HfEnvTaskTestBusiness/getProjectInfoByProjectId',
        queryById: '/HfEnvEntrustBusiness/queryById',
        saveCurve: '/HfEnvTaskTestBusiness/saveCurve'
      },
      equipCodeProps: '',
      selectRowIdProps: '',
      leftTreeChange: false,
      spinLoading: false,
      enlargementShow: false,//放大按钮默认不显示
      queryParams: {},
      chartsParams: [],
      entrustIds: [],
      collectionCurveData: [],
      settingCurveData: [],
      calcSettingResult: [],
      calcCollectionResult: [],
      calcAllResult: {},
      settingDataParams: [
        {"name": "温度设定值", "type": "Temperature_SV"},
        {"name": "湿度设定值", "type": "Humidity_SV"},
      ],
      defaultChartData: [['tagName', 'tagValue', 'TimeStamp']]
    }
  },
  methods: {
    enlargement() {
      this.$emit('open', this.calcAllResult)
    },
    moment,
    onDateChange(row) {
      this.queryParams.initialTimeRange = row
      this.queryParams.initialTime_startTime = new Date(row[0]).getTime() / 1000 || 0
      this.queryParams.initialTime_endTime = new Date(row[1]).getTime() / 1000 || 0
      this.queryParams.initialTime = row[0]
      this.initialTemTime = moment(row[0]).format('x')
      this.initialHumTime = moment(row[0]).format('x')
      this.handleSearch()
    },
    clearChart() {
      this.spinLoading = false
      setTimeout(() => {
        this.$echarts.init(document.getElementById('eCharts')).clear()
      }, 10)
    },
    // 根据项目id获取设定值
    getSettingCurveData() {
      return new Promise((resolve, reject) => {
        getAction(this.url.queryByProjectId, {projectIds: this.selectedRow[0].projectIds}).then(res => {
          if (res.code === 200 && Object.keys(res.data).length) {
            this.settingCurveData = res.data[0].abilityRequire
            resolve(this.settingCurveData)
          } else {
            reject('未查询到规划曲线数据')
          }
        })
      })
    },
    // 根据时间区间查询采集库数据
    getCollectionCurveData() {
      return new Promise((resolve, reject) => {
        let params = {
          ...this.queryParams,
          equipCode: this.equipCodeProps
        }
        delete params.initialTimeRange
        getAction(this.url.listAll, params).then((res) => {
          if (res.code === 200) {
            this.collectionCurveData = res.data
            resolve(this.collectionCurveData)
          } else {
            reject('未查询到采集曲线数据')
          }
        })
      })
    },
    initialSearcher() {
      this.initialTemTime = moment(this.queryParams.initialTime).format('x')//温度湿度初始时间
      this.initialHumTime = moment(this.queryParams.initialTime).format('x')//温度湿度初始时间
      this.initialTemperature = this.queryParams.initialTemperature || 25 // 初始温度
      this.initialHumidity = this.queryParams.initialHumidity || 30 // 初始湿度
    },
    handleSearch() {
      this.initialSearcher()
      this.spinLoading = true
      this.$nextTick().then(() => {
        let {initialTime, initialTemperature, initialHumidity, initialTimeRange} = this.queryParams
        if (!initialTime) {
          this.spinLoading = false
          return this.$message.warning('请选择初始时间')
        }
        if (!this.selectedRow.length) {
          this.spinLoading = false
          return this.$message.warning('请选择试验任务')
        }
        if (initialTimeRange && (initialTime || initialTemperature || initialHumidity)) {
          // 如果有了时间区间，并且还有温度或湿度或时间，那么将采集数据和设定数据合并展示
          let record = {data: this.defaultChartData, params: []}
          this.getSettingCurveData().then(result => {
            if (result && result.length) {
              this.splitByCurveType(result)
              record.data = record.data.concat(this.calcSettingResult)
              record.params = this.settingDataParams
              this.getCollectionCurveData().then(result1 => {
                if (result1 && Object.keys(result1).length && result1.data && result1.data.length && result1.params && result1.params.length) {
                  record.data = record.data.concat(result1.data)
                  record.params = record.params.concat(result1.params)
                } else {
                  this.$message.warning('未查询到采集曲线数据，为您展示规划曲线数据')
                }
                this.drawChartBefore(record)
              })
            } else {
              this.$message.warning('未查询到规划曲线数据，为您展示采集曲线数据')
              this.getCollectionCurveData().then(result1 => {
                if (result1 && Object.keys(result1).length && result1.data && result1.data.length && result1.params && result1.params.length) {
                  record.data = record.data.concat(result1.data)
                  record.params = record.params.concat(result1.params)
                  this.drawChartBefore(record)
                } else {
                  this.$message.warning('未查询到规划曲线数据和采集曲线数据')
                }
              })
            }
          }).catch(() => {
            this.clearChart()
            this.$message.warning('未查询到规划曲线数据')
          })
        } else if (initialTime || initialTemperature || initialHumidity) {
          // 如果只有温度，湿度，和时间，那么只查设定值
          this.getSettingCurveData().then(result => {
            if (result && result.length) {
              this.splitByCurveType(result)
              let record = {
                data: this.defaultChartData.concat(this.calcSettingResult),
                params: this.settingDataParams
              }
              this.drawChartBefore(record)
            } else {
              this.clearChart()
              this.$message.warning('未查询到规划曲线数据')
            }
          }).catch(() => {
            this.clearChart()
            this.$message.warning('未查询到规划曲线数据')
          })
        } else if (initialTimeRange) {
          // 如果仅仅只有时间区间，那么只查询采集数据
          this.getCollectionCurveData().then(result => {
            if (result && result.length) {
              let record = {
                data: this.defaultChartData.concat(result.data),
                params: result.params
              }
              this.spinLoading = false
              this.$nextTick(() => {
                this.drawChart(record)
                this.enlargementShow = true
              })
            } else {
              this.clearChart()
              this.$message.warning('未查询到采集曲线数据')
            }
          }).catch(() => {
            this.clearChart()
            this.$message.warning('未查询到采集曲线数据')
          })
        }
      })
    },
    drawChartBefore(record) {
      this.spinLoading = false
      this.$nextTick(() => {
        this.drawChart(record)
        this.enlargementShow = true
        this.calcAllResult = record
      })
    },
    handleReset() {
      this.queryParams = {}
      this.clearChart()
      this.initialCurveData()
    },
    splitByCurveType(list) {
      this.entrustOrTaskFlag = true
      try {
        this.isHighTemperature = list[0].abilityInfo.filter(item => item.paramName === '初始类型')[0].conditionTypeDesc === '1'
      } catch {
        this.isHighTemperature = true
      }
      let calcResult = []
      let currentDataSourceExtend = list.filter(item => item.type === 'stage')
      for (let i = 0; i < currentDataSourceExtend.length; i++) {
        let item = currentDataSourceExtend[i].abilityInfo
        try {
          this.loopNum = item.filter(v => +v.curveType === 1 && v.paramCode === 'qh07')[0].minValue
        } catch {
          this.loopNum = 1
        }
        calcResult = calcResult.concat(this.drawTemperatureCurve(item.filter(v => +v.curveType === 1)).result).concat(this.drawHumidityCurve(item.filter(v => +v.curveType === 2)).result)
      }
      this.calcSettingResult = calcResult
      this.initialCurveData()
    },
    initialCurveData() {
      this.initialTemTime = moment(0).format('x')//温度湿度初始时间
      this.initialHumTime = moment(0).format('x')//温度湿度初始时间
      this.initialTemperature = 25 // 初始温度
      this.initialHumidity = 30 // 初始湿度
      this.entrustOrTaskFlag = false
    },
    drawChart(record) {
      let that = this
      let myChart = this.$echarts.init(document.getElementById('eCharts'))
      let myDataset = []
      let mySeries = []
      let chartParams = record.params || []
      let chartData = record.data || []
      if (chartParams.length > 0) {
        chartParams.forEach(i => {
          myDataset.push(
            {
              id: i.type,
              fromDatasetId: 'dataset_raw',
              transform: {
                type: 'filter',
                config: {
                  and: [
                    {dimension: 'tagName', '=': i.type},
                  ]
                }
              }
            }
          )
          mySeries.push(
            {
              name: i.name,
              type: 'line',
              datasetId: i.type,
              showSymbol: false,
              encode: {
                x: 'TimeStamp',
                y: 'tagValue',
                itemName: 'TimeStamp',
                tooltip: ['tagValue']
              }
            }
          )
        })
      } else {
        myDataset = []
        mySeries = []
      }
      let option = {
        dataset: [
          {
            id: 'dataset_raw',
            source: chartData
          }
        ].concat(myDataset),
        title: {
          // text: '规划曲线'
        },
        tooltip: {
          trigger: 'axis'
        },
        grid: {
          bottom: 80
        },
        toolbox: {
          feature: {
            saveAsImage: {},
            dataZoom: {},
            mySaveQx: {
              show: true,
              title: '保存到曲线',
              // E:\projects\Hifar\tdm200-client\src\assets\signature-name.png
              // icon:"image://https://img95.699pic.com/element/40133/9422.png_300.png",
              icon: 'path://M814.805 128a51.179 51.179 0 0 1 51.179 51.179V844.01a51.179 51.179 0 0 1-51.179 51.157H201.173a51.179 51.179 0 0 1-51.178-51.157V179.179A51.179 51.179 0 0 1 201.173 128h613.654zM329.024 434.837a51.093 51.093 0 0 1-51.179-51.093V179.157h-76.672v664.854h613.76V179.179H738.22v204.48a51.179 51.179 0 0 1-51.179 51.178H329.024z m0-51.093h357.995V179.157H329.024v204.587z m357.91 204.501a25.557 25.557 0 1 1 0.085 51.072H329.024a25.536 25.536 0 1 1 0-51.072h357.91z',
              onclick() {
                if (that.selectedRow.length === 0) {
                  that.$message.info('请选择一项')
                  return
                }
                console.log("selectRow", that.selectedRow)
                let testCodes = that.selectedRow.map(item => item.testCode).join(',')
                let _this = this;
                that.$confirm({
                  title: '确认保存',
                  content: `是否保存到试验编号为${testCodes}的任务吗`,
                  onOk: function () {
                    that.saveCurve(_this)
                  }
                })
              }
            }
          },
          right: 40
        },
        dataZoom: [
          {
            type: 'inside',
            start: 0,
            end: 100
          },
          {
            start: 0,
            end: 100
          },
        ],
        legend: {},
        xAxis: {
          type: 'time',
          nameLocation: 'middle',
          axisLabel: {
            show: true,
            showMinLabel: true,
            showMaxLabel: true,
            formatter: (value) => {
              return moment(value).format('MM-DD hh:mm').replace(" ", '\n')
            }
          },
          boundaryGap: false,
        },
        yAxis: {
          // name: '试验值'
        },
        series: mySeries
      }
      myChart.clear()
      setTimeout(function () {
        myChart.setOption(option)
        window.onresize = function () {
          myChart.resize()
        }
      }, 200)
    },
    saveCurve(charts) {
      let base64Data = charts.api.getDataURL('png')
      // let file = dataURLtoFile(base64Data, '曲线图.png')
      let data = {
        file: base64Data,
        testIds: this.selectRowId.map(item => item.id).join(",")
      }
      console.log("data", data)
      postAction(this.url.saveCurve, data).then(res => {
        if (res.code == 200) {
          this.$message.success("保存成功")
        } else {
          this.$message.error("保存失败")
        }
      })
      // FIXME: 前端进行文件上传,暂未实现
      /*        tryUpload(file)
                .then(res => {
                  console.log('上传测功', res)
                  let data={
                    file:res,
                    testIds:this.selectRowId.map(item=>item.id).join(",")
                  }
                  console.log("data",data)
                  postAction(this.url.saveCurve,data).then(res=>{
                    if(res.code==200){
                      this.$message.success("保存成功")
                    }else{
                      this.$message.error("保存失败")
                    }
                  })
                })
                .catch(err => {
                  console.log('上传失败', err)
                })*/
    },
  }
}
</script>
<style lang="less" scoped>
.eCharts {
  width: 100%;
  height: 240px;
}

.spin-box {
  height: 230px;
  line-height: 230px;
  text-align: center;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 4px;
}

/deep/ .empty-box .ant-empty {
  height: 230px;
}
</style>