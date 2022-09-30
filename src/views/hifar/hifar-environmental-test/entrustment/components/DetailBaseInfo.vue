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
      {{ detailData.entrustNo || '--' }}
    </h-desc-item>
    <h-desc-item label='运行单号'>
      {{ detailData.entrustCode || '--' }}
    </h-desc-item>
    <h-desc-item label='委托状态'>
      <a-tag slot='content' :color='detailData.status | wtStatusColorFilter' class='status_tag'>
        {{ detailData.status | wtStatusFilter }}
      </a-tag>
    </h-desc-item>
    <h-desc-item :label='detailData.entrustType == 1 ? "委托日期" : "申请日期"'>
      {{
        detailData.entrustTime && detailData.entrustTime != 0
          ? moment(parseInt(detailData.entrustTime)).format('YYYY-MM-DD')
          : '--'
      }}
    </h-desc-item>
    <h-desc-item label='委托单类型'>
      {{ detailData.entrustType | entrustTypeFilter }}
    </h-desc-item>
    <h-desc-item label='委托单位'>
      {{ detailData.custName || '--' }}
    </h-desc-item>
    <h-desc-item :label='detailData.entrustType == 1 ? "联系人" : "申请人"'>
      {{ detailData.linkName || '--' }}
    </h-desc-item>
    <h-desc-item label='联系方式'>
      {{ detailData.linkMobile || '--' }}
    </h-desc-item>
    <h-desc-item label='单位地址' v-if="detailData.entrustType == 1">
      {{ detailData.custAddress || '--' }}
    </h-desc-item>
    <h-desc-item label='密级'>
      {{ detailData.secretLevelCode_dictText || '--' }}
    </h-desc-item>
    <template v-if="detailData.entrustType == 1">
      <h-desc-item label='样品制造单位'>
        {{ detailData.sampleMakeUnit || '--' }}
      </h-desc-item>
      <h-desc-item label='样品状态'>
        {{ detailData.sampleStatus_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='样品提供方式'>
        {{ detailData.sampleProvisionMethod_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='样品处置方式'>
        {{ detailData.sampleDisposeMethod_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='试验目的'>
        {{ detailData.testPurpose_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='进度要求'>
        {{ detailData.progressRequire_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='性能测试'>
        {{ detailData.performanceTest_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='检测照片'>
        {{ detailData.testPicture_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='报告形式'>
        {{ detailData.reportForm_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='报告密级'>
        {{ detailData.reportSecretLevel_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='报告份数'>
        {{ detailData.reportNum || '--' }}
      </h-desc-item>
      <h-desc-item label='报告领取方式'>
        {{ detailData.reportCollectionMethod_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='委托人'>
        {{ detailData.entrustPerson || '--' }}
      </h-desc-item>
      <h-desc-item label='委托人手机号'>
        {{ detailData.entrustPersonPhone || '--' }}
      </h-desc-item>
    </template>
    <template v-if="detailData.entrustType == 2">
      <h-desc-item label='工作令号'>
        {{ detailData.workOrderNo || '--' }}
      </h-desc-item>
      <h-desc-item label='产品名称'>
        {{ detailData.productName || '--' }}
      </h-desc-item>
      <h-desc-item label='试验目的'>
        {{ detailData.testPurpose_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='要求完成时间'>
        {{
          detailData.requireCompletionTime && detailData.requireCompletionTime != 0
            ? moment(parseInt(detailData.requireCompletionTime)).format('YYYY-MM-DD HH:mm:ss')
            : '--'
        }}
      </h-desc-item>
      <h-desc-item label='样品处置方式'>
        {{ detailData.sampleDisposeMethod_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='性能测试'>
        {{ detailData.performanceTest_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='测试地点'>
        {{ detailData.testAddress_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='检测报告'>
        {{ detailData.testReport_dictText || '--' }}
      </h-desc-item>
      <h-desc-item label='试验依据'>
        {{ detailData.testEvidence || '--' }}
      </h-desc-item>
      <h-desc-item label='试验要求'>
        {{ detailData.testRequire || '--' }}
      </h-desc-item>
    </template>
    <h-desc-item label='创建人'>
      {{ detailData.createUserName || '--' }}
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
      {{ detailData.remarks || '--' }}
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