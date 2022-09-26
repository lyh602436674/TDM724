<template>
  <div style="height: 100%">
    <vxe-table ref='workTable'
               :data='tableData'
               @cell-click="handleCellClick"
               height="100%"
               size="medium"
               autoResize round border stripe
               :edit-config="{
                  trigger: 'click',
                  mode: 'cell',
                  activeMethod: ()=> this.tableEdit
                }"
               keep-source
               show-overflow>
      <vxe-table-column field='date' title="日期" align="center">
        <div slot='default' slot-scope='{row, rowIndex}'>
          <div style="display: flex;width: 100%;justify-content: center">
            <div class="date" style="padding:0 5px">{{ row.date }}</div>
            <div class="week" style="padding:0 5px">{{ row.week }}</div>
          </div>
        </div>
      </vxe-table-column>
      <template>
        <vxe-table-column v-if="tableEdit" :edit-render="{}" field="user" align="center" title="排班">
          <template #edit="{row,rowIndex,column}">
            <a-select v-model="row.user" show-search
                      @change="(e) => handleUserChange(e,row,rowIndex,[{key:'concatWay',value:'mobile'},{key:'userName',value:'idName'}])"
                      :getPopupContainer="(node)=> node.parentNode" style="width: 100%" :filter-option="filterOption">
              <a-select-option v-for="item in userList" :key="item.id" :value="item.id">
                {{ item.idName }}
              </a-select-option>
            </a-select>
          </template>
          <template v-slot="{row}">
            <span>{{ row.userName }}</span>
          </template>
        </vxe-table-column>
        <vxe-table-column v-else field="userName" align="center" title="排班"/>
      </template>
      <vxe-table-column
        align="center"
        field='concatWay'
        title="联系方式"
      />
      <template>
        <vxe-table-column v-if="tableEdit" :edit-render="{}" field="realityUser" align="center" title="实际">
          <template #edit="{row,rowIndex}">
            <a-select v-model="row.realityUser" show-search
                      @change="(e) => handleUserChange(e,row,rowIndex,[{key:'realityConcatWay',value:'mobile'},{key:'realityUserName',value:'idName'}])"
                      :getPopupContainer="(node)=> node.parentNode" style="width: 100%" :filter-option="filterOption">
              <a-select-option v-for="item in userList" :key="item.id" :value="item.id">{{
                  item.idName
                }}
              </a-select-option>
            </a-select>
          </template>
          <template v-slot="{row}">
            <span>{{ row.realityUserName }}</span>
          </template>
        </vxe-table-column>
        <vxe-table-column v-else field="realityUserName" align="center" title="实际"/>
      </template>
      <vxe-table-column
        field='realityConcatWay'
        title="联系方式"
        align="center"
      />
      <template>
        <vxe-table-column v-if="tableEdit" :edit-render="{}" field="user1" align="center" title="排班">
          <template #edit="{row,rowIndex}">
            <a-select v-model="row.user1" show-search autoFocus
                      @change="(e) => handleUserChange(e,row,rowIndex,[{key:'concatWay1',value:'mobile'},{key:'userName1',value:'idName'}])"
                      :getPopupContainer="(node)=> node.parentNode" style="width: 100%" :filter-option="filterOption">
              <a-select-option v-for="item in userList" :key="item.id" :value="item.id">{{
                  item.idName
                }}
              </a-select-option>
            </a-select>
          </template>
          <template v-slot="{row}">
            <span>{{ row.userName1 }}</span>
          </template>
        </vxe-table-column>
        <vxe-table-column v-else field="userName1" align="center" title="排班"/>
      </template>
      <vxe-table-column
        field='concatWay1'
        title="联系方式"
        align="center"
      />
      <template>
        <vxe-table-column v-if="tableEdit" :edit-render="{}" field="realityUser1" align="center" title="实际">
          <template #edit="{row,rowIndex}">
            <a-select v-model="row.realityUser1" show-search
                      @change="(e) => handleUserChange(e,row,rowIndex,[{key:'realityConcatWay1',value:'mobile'},{key:'realityUserName1',value:'idName'}])"
                      :getPopupContainer="(node)=> node.parentNode" style="width: 100%" :filter-option="filterOption">
              <a-select-option v-for="item in userList" :key="item.id" :value="item.id">{{
                  item.idName
                }}
              </a-select-option>
            </a-select>
          </template>
          <template v-slot="{row}">
            <span>{{ row.realityUserName1 }}</span>
          </template>
        </vxe-table-column>
        <vxe-table-column v-else field="realityUserName1" align="center" title="实际"/>
      </template>
      <vxe-table-column
        field='realityConcatWay1'
        title="联系方式"
        align="center"
      />
    </vxe-table>
  </div>
</template>

<script>
export default {
  name: "userTable",
  props: {
    tableData: {
      type: Array,
      default: () => []
    },
    userList: {
      type: Array,
      default: () => []
    },
    tableEdit: {
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
      localTableData: [],
    }
  },
  methods: {
    handleUserChange(id, row, rowIndex, setData) {
      for (let i = 0; i < setData.length; i++) {
        this.$set(this.localTableData[rowIndex], setData[i].key, this.userList.filter(item => item.id === id)[0][setData[i].value])
      }
    },
    handleCellClick({row, rowIndex, column}) {

    },
    filterOption(input, option) {
      return (
        option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
      );
    }
    ,
  }
}
</script>

<style scoped>

</style>