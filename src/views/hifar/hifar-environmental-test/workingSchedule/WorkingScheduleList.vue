<template>
  <div ref='workingSchedule' class='workingSchedule'>
    <h-card fixed bordered>
      <template slot='title'>排班计划表</template>
      <h-tabs fixed slot='content' :activeKey="activeKey" type="card" @change="handleTabsChange">
        <a-button v-if="activeKey==='1'" slot="tabBarExtraContent" type="primary" @click="handleSave">保存</a-button>
        <a-tab-pane key="1" tab="当月排班">
          <working-schedule-table ref="monthlyTable" :table-data="monthlyTableData" :userList="userList"
                                  tableEdit table-data-name="monthlyTableData"></working-schedule-table>
        </a-tab-pane>
        <a-tab-pane key="2" tab="历史排班" :forceRender="true">
          <working-schedule-table :table-data="historyTableData" table-data-name="historyTableData" :userList="userList"></working-schedule-table>
        </a-tab-pane>
      </h-tabs>

    </h-card>
  </div>
</template>

<script>
import {getAction, postAction} from '@/api/manage'
import WorkingScheduleTable from "./WorkingScheduleTable";

export default {
  name: "WorkingScheduleList",
  components: {WorkingScheduleTable},
  data() {
    return {
      activeKey: "1",
      historyTableData: [],
      monthlyTableData: [],
      columns: [
        {
          title: '日期',
          align: 'center',
          dataIndex: 'date',
          scopedSlots: {customRender: "date"}
        },
        {
          title: '排班',
          align: 'center',
          dataIndex: 'workingSchedule',
          scopedSlots: {customRender: "workingSchedule"}
        },
        {
          title: '联系方式',
          align: 'center',
          dataIndex: 'concatWay',
          scopedSlots: {customRender: "concatWay"}
        },
        {
          title: '实际',
          align: 'center',
          dataIndex: 'practical',
          scopedSlots: {customRender: "practical"}
        },
        {
          title: '排班',
          align: 'center',
          dataIndex: 'workingSchedule1',
          scopedSlots: {customRender: "workingSchedule1"}
        },
        {
          title: '联系方式',
          align: 'center',
          dataIndex: 'concatWay1',
          scopedSlots: {customRender: "concatWay1"}
        },
        {
          title: '实际',
          align: 'center',
          dataIndex: 'practical1',
          scopedSlots: {customRender: "practical1"}
        },
      ],
      queryParams: {},
      url: {
        list: "/DutyRoster/getDutyRosterDate",
        user: "/DutyRoster/showUser",
      },
      userList: [],

    }
  },
  activated() {
    this.loadData()
    this.getUserList()
  },
  methods: {
    loadData() {
      postAction(this.url.list).then((res) => {
        if (res.code === 200) {
          this.monthlyTableData = res.data.data
          this.historyTableData = res.data.data
        }
      })
    },
    handleSave() {
      let monthlyTableData = this.$refs.monthlyTable.$refs.workTable.getData()
      console.log(monthlyTableData, 'monthlyTableData')
    },
    handleTabsChange(value) {
      this.activeKey = value
    },
    getUserList() {
      getAction(this.url.user).then((res) => {
        if (res.code === 200) {
          this.userList = res.data
        } else {
          this.$message.warning('获取用户数据失败')
        }
      })
    },
  }
}
</script>

<style scoped lang="less">
/deep/ .ant-select-selection-selected-value {
  width: 100%;
}
</style>