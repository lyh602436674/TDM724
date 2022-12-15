<!--
 * @Author: 雷宇航
 * @Date: 2022-03-31 11:34:13
 * @LastEditTime: 2022-04-12 18:13:39
 * @LastEditors: 雷宇航
 * @Description:
 * @FilePath: \tdm200-client\tdm200-client\src\views\hifar\hifar-environmental-test\largeScreen\EquipDetailList.vue
-->
<template>
  <div class="equipDetailList">
    <div class="title">监控列表</div>
    <div class="content">
      <div class="content-table">
        <div class="table-header">
          <div class="table-header-item" v-for="(item, index) in columns" :key="index" :style="{ width: item.width }">
            {{ item.title }}
          </div>
        </div>
        <div class="table-body">
          <!--          <h-auto-scroll :data="dataSource" class="table-body-scroll" :class-option="classOption">-->
          <div v-for="(item, index) in dataSource" :key="index + '-tr'" :style="rowStyle"
               class="table-body-row">
            <div v-for="(v, i) in columns" :key="index + i + 'td'" :style="{ width: v.width }" class="table-body-col">
              <template v-if="v.key === 'index'">
                {{ 1 + index }}
              </template>
              <template v-else-if="v.key !== 'status'">
                <a-tooltip :title="item[v.key]">
                  <span>{{ item[v.key] }}</span>
                </a-tooltip>
              </template>
              <template v-else>
                <div :style="{backgroundColor:colorList[item.statusNum-1]}" class="table-body-col-status">
                  {{ item[v.key] ? item[v.key] : '--' }}
                </div>
              </template>
            </div>
          </div>
          <!--          </h-auto-scroll>-->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {getAction} from '@/api/manage'
import moment from 'moment'
import HAutoScroll from "@comp/HAutoScroll/HAutoScroll";
import {cloneDeep} from 'lodash'

export default {
  name: 'EquipDetailList',
  components: {HAutoScroll},
  description: '设备信息列表页面',
  mounted() {
    window.addEventListener('resize', () => {
      this.getTableBodyRowHeight()
    })
  },
  props: {
    timerStep: {
      type: Number,
      default: 0
    }
  },
  computed: {
    classOption() {
      return {
        singleHeight: this.rowStyle.height,
        waitTime: 5000,
      }
    }
  },
  data() {
    return {
      rowStyle: {},
      columns: [
        {title: '序号', width: '6%', key: 'rowSort'},
        {title: '设备名称', width: '20%', key: 'equipName'},
        {title: '温度', width: '6%', key: 'temperatureSV'},
        {title: '湿度', width: '6%', key: 'humiditySV'},
        {title: '项目名称', width: '12%', key: 'unitName'},
        {title: '委托单位', width: '22%', key: 'custName'},
        {title: '预计结束时间', width: '18%', key: 'predictEndTime'},
        {title: '状态', width: '10%', key: 'status'},
      ],
      dataSource: [],
      colorList: ['#26ad53', '#ff7800', '#ff0000',],
      url: {
        list: '/LargeScreenDisplay/taskMonitor',
      },
      pageTimer: null,
      pageSize: 2,
      timerIndex: 1,
      dataSourceWatch: [],
    }
  },
  watch: {
    dataSourceWatch(newData, oldData) {
      if (!(JSON.stringify(newData) === JSON.stringify(oldData))) {
        this.autoPageTurning(newData)
      }
    },
  },
  created() {
    // this.loadData()
  },
  methods: {
    getTableBodyRowHeight() {
      const tableBodyHeight = document.getElementsByClassName('table-body')[0]
      const height = tableBodyHeight.offsetHeight / 10 + "px"
      this.rowStyle = {height, lineHeight: height}
    },
    moment,
    loadData() {
      getAction(this.url.list).then((res) => {
        if (res.code === 200) {
          let dataSource = this.formatDataSource(res.data)
          let {pageSize} = this
          if (dataSource.length <= pageSize) {
            this.dataSource = dataSource
          } else {
            this.dataSourceWatch = dataSource
          }
        }
      })
    },
    formatDataSource(dataSource) {
      return dataSource.map((item) => {
        return {
          ...item,
          statusNum: item.status,
          status: this.deviceStatusFilter(item.status),
          predictEndTime: item.predictEndTime ? moment(+item.predictEndTime).format('YYYY-MM-DD HH:mm:ss') : ''
        }
      }).sort((a, b) => a.rowSort - b.rowSort)
    },
    autoPageTurning(dataSource) {
      let {pageSize} = this
      this.dataSource = dataSource.slice(0, pageSize)
      let extendData = cloneDeep(dataSource)
      let pageTotal = Math.ceil(dataSource.length / pageSize)
      let result = []
      for (let i = 0; i < pageTotal; i++) {
        result.push(extendData.slice(i * pageSize, (i + 1) * pageSize))
      }
      if (this.pageTimer) clearInterval(this.pageTimer)
      this.pageTimer = setInterval(() => {
        this.dataSource = result[this.timerIndex]
        this.timerIndex++
        if (this.timerIndex >= pageTotal) this.timerIndex = 0
      }, 10000)
    },
    deviceStatusFilter(status) {
      let s = parseInt(status)
      switch (s) {
        case 1:
          return '占用'
        case 2:
          return '待机'
        default:
          return '--'
      }
    },
  },
}
</script>

<style scoped lang="less">


.equipDetailList {
  .title {
    width: 100%;
    text-align: center;
    color: #00f6ff;
    font-size: 0.1rem;
    padding: 0.052rem 0 /* 20/384 */;
    background: url('./image/bottom_line.png') bottom no-repeat;
  }

  .content {
    width: 100%;
    height: calc(100% - 0.252rem);
    padding: 0.05rem 0.2rem;
    overflow: hidden;

    &-table {
      width: 100%;
      height: 100%;
      border: 0.005rem solid rgb(13, 72, 224);
      // border-bottom: none;
      overflow: hidden;

      .table-header {
        height: 0.167rem /* 64/384 */;
        width: 100%;
        overflow: hidden;
        border-bottom: 0.005rem solid rgb(13, 72, 224) !important;
        display: flex;

        &-item {
          height: 0.167rem /* 64/384 */;
          line-height: 0.161rem /* 62/384 */;
          text-align: center;
          border-right: 0.005rem solid rgb(13, 72, 224);
          font-size: 0.083rem /* 32/384 */;

          &:last-child {
            border-right: none;
          }
        }
      }

      .table-body {
        width: 100%;
        height: calc(100% - 0.167rem);
        overflow: hidden;
        display: flex;
        flex-direction: column;

        &-scroll {
          width: 100%;
          height: 100%;
          overflow: hidden;
        }

        &-row {
          width: 100%;
          height: 0.167rem /* 64/384 */;
          line-height: 0.167rem /* 64/384 */;
          text-align: center;
          font-size: 0.083rem /* 32/384 */;
          border-bottom: 0.005rem solid rgb(13, 72, 224);
          display: flex;
        }

        &-col {
          height: 100% /* 64/384 */;
          padding: 0 0.026rem /* 10/384 */;
          overflow: hidden;
          white-space: nowrap;
          text-overflow: ellipsis;
          border-right: 0.005rem solid rgb(13, 72, 224);

          &-status {
            display: inline-block;
            width: 80%;
            height: .12rem /* 46/384 */;
            line-height: .11rem;
            text-align: center;
            margin-top: .023rem /* 9/384 */;
            border-radius: .013rem /* 5/384 */;
          }

          &:last-child {
            border-right: none;
          }
        }
      }
    }
  }
}
</style>