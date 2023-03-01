<template>
  <div style="height: 100%">
    <vxe-table ref='workTable'
               :data='tableData'
               height="100%"
               :loading="tableLoading"
               size="medium"
               autoResize round border stripe
               keep-source
               show-overflow>
      <vxe-table-column field='dutyTime' title="日期" align="center">
        <div slot='default' slot-scope='{row, rowIndex}'>
          <div style="display: flex;width: 100%;justify-content: center">
            <div class="time" style="padding:0 5px">
              {{ row.dutyTime && moment(+row.dutyTime).format('YYYY-MM-DD') || '--' }}
            </div>
            <div class="week" style="padding:0 5px">{{ row.week }}</div>
          </div>
        </div>
      </vxe-table-column>
      <vxe-table-column align="center" field="dayShift" title="白班">
        <template v-slot="{row}">
          <a-button v-model="row.dayShift" :type="row.dayShift ? 'primary' : null" style="width:100%"
                    @click="handleShift(row,'dayShift')">到岗
          </a-button>
        </template>
      </vxe-table-column>
      <vxe-table-column
        field='dayShiftContact'
        title="联系人/联系方式"
        align="center"
      />
      <vxe-table-column
        :visible="false"
        field='dayShiftContactId'
        align="center"
      />
      <vxe-table-column align="center" field="nightShift" title="夜班">
        <template v-slot="{row}">
          <a-button v-model="row.nightShift" :type="row.nightShift ? 'primary' : null" style="width:100%"
                    @click="handleShift(row,'nightShift')">到岗
          </a-button>
        </template>
      </vxe-table-column>
      <vxe-table-column
        field='nightShiftContact'
        title="联系人/联系方式"
        align="center"
      />
      <vxe-table-column
        :visible="false"
        align="center"
        field='nightShiftContactId'
      />
    </vxe-table>
  </div>
</template>

<script>
import moment from "moment";

export default {
  name: "WorkingScheduleTable",
  props: {
    tableData: {
      type: Array,
      default: () => []
    },
    tableEdit: {
      type: Boolean,
      default: false
    },
    tableLoading: {
      type: Boolean,
      default: false
    },
    tableDataName: {
      type: String,
      default: 'tableData',
      required: true
    },
  },
  watch: {
    tableData: {
      immediate: true,
      deep: true,
      handler(val) {
        this.localTableData = val
      },
    },
  },
  data() {
    return {
      moment,
      localTableData: [],
      userInfo: this.$store.getters.userInfo,
    }
  },
  methods: {
    handleShift(row, field) {
      if (!this.tableEdit) return
      if (!row[field]) {
        this.$set(row, field, 1)
      } else {
        this.$set(row, field, 0)
      }
      let contact = this.userInfo.idName + ' ' + this.userInfo.mobile
      let userId = this.userInfo.id
      if (field === 'dayShift') {
        this.$set(row, 'dayShiftContact', contact)
        this.$set(row, 'dayShiftContactId', userId)
      } else if (field === 'nightShift') {
        this.$set(row, 'nightShiftContact', contact)
        this.$set(row, 'nightShiftContactId', userId)
      }
      this.$emit('change', row)
    },
  }
}
</script>

<style scoped>

</style>