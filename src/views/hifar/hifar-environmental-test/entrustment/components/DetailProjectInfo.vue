<!--
 * @Author: 赵峰
 * @Date: 2021-09-18 09:56:30
 * @Descripttion: 委托管理--详情-项目信息
 * @FilePath: \hifar-platform-client\src\views\hifar\hifar-environmental-test\entrustment\components\DetailProjectInfo.vue
-->
<template>
  <h-desc :bordered="false" size="small" title="项目信息">
    <h-card v-for="(item, index) in projectInfo" :key="index" style="margin-bottom: 10px">
      <div slot="title">{{ item.unitName }}</div>
      <h-desc slot="content" labelWidth="130px" size="small">
        <h-desc-item label="项目名称">
          {{ item.unitName ? item.unitName : '--' }}
        </h-desc-item>
        <h-desc-item label="试验名称">
          {{ item.testName ? item.testName : '--' }}
        </h-desc-item>
        <h-desc-item label="试验标准">
          {{ item.standardCode + item.standardName || '--' }}
        </h-desc-item>
<!--        <h-desc-item label="预计开始时间">-->
<!--          {{-->
<!--            item.expectStartTime && item.expectStartTime != 0-->
<!--              ? moment(parseInt(item.expectStartTime)).format('YYYY-MM-DD HH:mm')-->
<!--              : '&#45;&#45;'-->
<!--          }}-->
<!--        </h-desc-item>-->
        <h-desc-item label="预计时长（h）">
          {{ item.predictDuration ? item.predictDuration + '小时' : '--' }}
        </h-desc-item>
<!--        <h-desc-item label="试验部门">-->
<!--          {{ item.workName ? item.workName : '&#45;&#45;' }}-->
<!--        </h-desc-item>-->
<!--        <h-desc-item label="试验项目试件">-->
<!--          {{ item.pieceNos ? item.pieceNos : '&#45;&#45;' }}-->
<!--        </h-desc-item>-->
        <h-desc-item label="总号">
          {{ item.sumMark ? item.sumMark : '--' }}
        </h-desc-item>
        <h-desc-item label="分号">
          {{ item.separateMark ? item.separateMark : '--' }}
        </h-desc-item>
        <h-desc-item v-if="item.needProcess == 1" :span="2" label="加工要求">
          {{ item.processRequire ? item.processRequire : '--' }}
        </h-desc-item>
        <h-desc-item :span="4" label="试验依据">
          {{ item.checkRequire ? item.checkRequire : '--' }}
        </h-desc-item>
        <h-desc-item :span="4" label="其他条件">
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
      </h-desc>
      <h-collapse
        v-if="item.curveUrl"
        :activeKey="1" :extraShow="false" style="margin-top: 10px" title="规划曲线">
        <div slot="content">
          <img :src="item.curveUrl" alt="规划曲线" height="90%" width="100%">
        </div>
      </h-collapse>
      <h-collapse :activeKey="1" :extraShow="false" style="margin-top: 10px" title="试验条件要求">
        <template slot="content">
          <template v-if="!item.abilityRequire.length"><a-empty style="width: 100%"></a-empty></template>
          <a-tabs v-else :default-active-key="0">
            <template v-for="(proItem,itemIndex) in item.abilityRequire">
              <a-tab-pane :key="itemIndex" :tab="proItem.type === 'stage' ? proItem.title + itemIndex : proItem.title">
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
                          row.conditionTypeDesc === '1' ? '先高温' : row.conditionTypeDesc === '2' ? '先低温' : '--'
                        }}</span>
                      <span v-else>{{ row.conditionTypeDesc }}</span>
                    </template>
                  </vxe-table-column>
                </vxe-table>
              </a-tab-pane>
            </template>
          </a-tabs>
        </template>
      </h-collapse>
    </h-card>
  </h-desc>
</template>

<script>
import moment from 'moment'
import mixin from '@/views/hifar/mixin.js'
import {downloadFile, getFileAccessHttpUrl} from "@api/manage";

export default {
  data() {
    return {
      moment,
      projectInfo: [],
      newArr: [],
    }
  },
  mixins: [mixin],
  props: {
    projectInfoArr: [Array],
  },
  watch: {
    projectInfoArr: {
      deep: true,
      immediate: true,
      handler(val) {
        if (val) {
          this.projectInfo = val
          this.projectInfo.forEach((item) => {
            item.pieceDetail.forEach((tt) => {
              this.newArr = tt.productAlias + '-' + tt.pieceNo
            })
          })
        }
      },
    },
  },

  created() {
  },
  methods: {
    handleDownload(filePath, fileName) {
      let fileAccessUrl = getFileAccessHttpUrl(filePath)
      downloadFile(fileAccessUrl, fileName)
    },
  },
}
</script>

<style lang="less" scoped>
.url-list {
  display: flex;
  align-items: center;
  margin-bottom: 10px;

  span {
    margin-right: 10px;
  }

}
</style>