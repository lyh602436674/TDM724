<!--
 * @Author: 赵峰
 * @Date: 2021-09-18 09:56:20
 * @LastEditTime: 2021-11-16 14:27:22
 * @LastEditors: 马潭龙
 * @Descripttion: 委托信息详情页面
 * @FilePath: \hifar-platform-client\src\views\hifar\hifar-environmental-test\entrustment\components\DetailBaseInfo.vue
-->
<template>
  <h-desc labelWidth='130px' size='small' title='委托信息'>
    <h-desc-item label='委托单号'>
      {{ detailData.entrustCode ? detailData.entrustCode : '--' }}
    </h-desc-item>
    <h-desc-item label='委托状态'>
      <a-tag slot='content' :color='detailData.status | wtStatusColorFilter' class='status_tag'>
        {{ detailData.status | wtStatusFilter }}
      </a-tag>
    </h-desc-item>
    <h-desc-item label='委托日期'>
      {{
        detailData.entrustTime && detailData.entrustTime != 0
          ? moment(parseInt(detailData.entrustTime)).format('YYYY-MM-DD')
          : '--'
      }}
    </h-desc-item>
    <h-desc-item label='委托单类型'>
      {{ detailData.entrustType | entrustTypeFilter }}
    </h-desc-item>
    <h-desc-item label='送试单位'>
      {{ detailData.custName ? detailData.custName : '--' }}
    </h-desc-item>
    <h-desc-item label='联系人'>
      {{ detailData.linkName ? detailData.linkName : '--' }}
    </h-desc-item>
    <h-desc-item label='联系方式'>
      {{ detailData.linkMobile ? detailData.linkMobile : '--' }}
    </h-desc-item>
    <h-desc-item label='客户地址' v-if="detailData.entrustType == 2">
      {{ detailData.custAddress ? detailData.custAddress : '--' }}
    </h-desc-item>
    <h-desc-item label='任务编码'>
      {{ detailData.outSourceCode ? detailData.outSourceCode : '--' }}
    </h-desc-item>
    <h-desc-item label='试验性质'>
      {{ detailData.testPropertyName ? detailData.testPropertyName : '--' }}
    </h-desc-item>
    <h-desc-item label='密级'>
      {{ detailData.secretLevelCode_dictText ? detailData.secretLevelCode_dictText : '--' }}
    </h-desc-item>
    <h-desc-item label='是否外包'>
      <span v-if="detailData.isExternalManage == 1" slot="content" style="color:red">是</span>
      <span v-else slot="content" style="color:green">否</span>
    </h-desc-item>
    <h-desc-item v-if="detailData.isExternalManage == 1" label='外包单位'>
      {{ detailData.outsourcingUnit ? detailData.outsourcingUnit : '--' }}
    </h-desc-item>
    <h-desc-item label='是否出报告'>
      {{ detailData.isReport == 1 ? '是' : detailData.isReport == 2 ? '否' : '--' }}
    </h-desc-item>
    <h-desc-item :label='detailData.entrustType === "1" ? "委托方批准人" : "委托方代表"'>
      {{ detailData.entrustApproval ? detailData.entrustApproval : '--' }}
    </h-desc-item>
    <template v-if="detailData.entrustType == 2">
      <h-desc-item label='是否允许对试品进行拍照'>
        {{ detailData.isPhotographByPiece == 1 ? '是' : detailData.isPhotographByPiece == 2 ? '否' : '--' }}
      </h-desc-item>
      <h-desc-item label='是否需要附试验原始数据'>
        {{ detailData.isNeedOriginalData == 1 ? '是' : detailData.isNeedOriginalData == 2 ? '否' : '--' }}
      </h-desc-item>
    </template>
    <h-desc-item v-if="detailData.entrustType==='2'" label='报告交付进度要求'>
      {{
        detailData.progressRequire_dictText ? detailData.progressRequire_dictText : '--'
      }}
    </h-desc-item>
    <h-desc-item v-if="detailData.entrustType==='2'" label='结算方式'>
      {{ detailData.settlementType_dictText ? detailData.settlementType_dictText : '--' }}
    </h-desc-item>

    <h-desc-item label='创建人'>
      {{ detailData.createUserName ? detailData.createUserName : '--' }}
    </h-desc-item>
    <h-desc-item label='创建时间'>
      {{
        detailData.createTime && detailData.createTime != 0
          ? moment(parseInt(detailData.createTime)).format('YYYY-MM-DD HH:mm:ss')
          : '--'
      }}
    </h-desc-item>
    <h-desc-item :span='4' label='委托单附件'>
      <div slot='content'>
        <template v-if='detailData.attachInfo && detailData.attachInfo.length'>
          <div v-for='(item, index) in detailData.attachInfo' :key='index' class='url-list'>
            <span>{{ index + 1 }}、{{ item.fileName }}</span>
            <a-button icon='download' size='small' type='primary' @click='handleDownload(item.filePath, item.fileName)'>
              下载
            </a-button>
          </div>
        </template>
        <span v-else>暂无附件</span>
      </div>
    </h-desc-item>
    <h-desc-item :span='6' label='备注'>
      {{ detailData.remarks ? detailData.remarks : '--' }}
    </h-desc-item>
  </h-desc>
</template>

<script>
import moment from 'moment'
import mixin from '@/views/hifar/mixin.js'
import {downloadFile, getFileAccessHttpUrl} from '@/api/manage'

export default {
  mixins: [mixin],
  props: {
    detailDataObj: {
      type: Object,
      default: () => {
      }
    }
  },
  watch: {
    detailDataObj: {
      immediate: true,
      handler(val) {
        if (val) {
          this.detailData = val
        }
      }
    }
  },

  data() {
    return {
      moment,
      detailData: {}
    }
  },

  methods: {
    handleDownload(filePath, fileName) {
      let fileAccessUrl = getFileAccessHttpUrl(filePath)
      downloadFile(fileAccessUrl, fileName)
    },
  }
}
</script>
<style lang='less' scoped>
.url-list {
  display: flex;
  align-items: center;
  margin-bottom: 10px;

  span {
    margin-right: 10px;
  }
}
</style>