<!--
 * @Author: 赵峰
 * @Date: 2021-09-18 09:53:53
 * @LastEditTime: 2021-11-18 18:06:34
 * @LastEditors: 赵峰
 * @Descripttion: 委托单详情页面组件
 * @FilePath: \hifar-platform-client\src\views\hifar\hifar-environmental-test\entrustment\components\EntrustDetail.vue
-->
<template>
  <div class="detail-containter">
    <!-- 委托信息 -->
    <detail-base-info :detailDataObj="detailData"></detail-base-info>
    <!--试品信息-->
    <h-desc labelWidth="120px" title="试品信息">
      <a-table
        :columns="columns[detailData.entrustType]"
        :dataSource="detailData.pieceInfo"
        :pagination="false"
        bordered
        rowKey="id"
        size="small"
        style="width: 100%; height: 100%"
      ></a-table>
    </h-desc>
    <!-- 项目信息 -->
    <detail-project-info
      :projectInfoArr="detailData && detailData.projectInfo ? detailData.projectInfo : []"
      style="margin-top: 15px"
    ></detail-project-info>
  </div>
</template>

<script>
import {postAction} from '@/api/manage'
import DetailBaseInfo from './DetailBaseInfo.vue'
import DetailProjectInfo from './DetailProjectInfo.vue'

export default {
  components: {
    DetailBaseInfo,
    DetailProjectInfo,
  },
  props: {
    id: {
      type: String,
      default: '',
    },
    detailData: {
      type: Object,
      default: () => {
      },
    },
  },
  watch: {
    id(val) {
      if (val) {
        this.show(this.id)
      }
    },
  },
  data() {
    return {
      activeKeyArr: '1',
      url: {
        detailById: '/HfEnvEntrustBusiness/queryById',
      },
      columns: {
        1: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: 'center',
            customRender: function (t, r, index) {
              return parseInt(index) + 1
            },
          },
          {
            title: '试品名称',
            dataIndex: 'productName',
          },
          {
            title: '试品工号',
            dataIndex: 'productCode',
          },
          {
            title: '试品代号',
            dataIndex: 'productAlias',
          },
          {
            title: '试品编号',
            dataIndex: 'pieceNo'
          }
        ],
        2: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: 'center',
            customRender: function (t, r, index) {
              return parseInt(index) + 1
            },
          },
          {
            title: '试品名称',
            dataIndex: 'productName',
          },
          {
            title: '试品型号',
            dataIndex: 'productAlias',
          },
          {
            title: '试品数量',
            dataIndex: 'pieceNum',
          },
          {
            title: '试品编号',
            dataIndex: 'pieceNo'
          }
        ],
      },
    }
  },
  created() {
    if (this.id) {
      this.show(this.id)
    }
  },
  methods: {
    show(id) {
      let url = this.url.detailById
      postAction(url, {id: id}).then((res) => {
        if (res.code == 200) {
          this.detailData = res.data
        }
      })
    },
  },
}
</script>
<style lang='less' scoped>
</style>