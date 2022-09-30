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
          {{ item.unitName || '--' }}
        </h-desc-item>
        <h-desc-item label="试验名称">
          {{ item.testName || '--' }}
        </h-desc-item>
        <h-desc-item label="试验标准">
          {{ item.standardCode + item.standardName || '--' }}
        </h-desc-item>
        <h-desc-item :span="4" label="试验依据">
          {{ item.checkRequire || '--' }}
        </h-desc-item>
        <h-desc-item :span="4" label="其他条件">
          {{ item.testCondition || '--' }}
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