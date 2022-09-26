<!--
 * @Author: 赵峰
 * @Date: 2021-11-05 10:40:54
 * @LastEditTime: 2021-11-05 13:49:35
 * @LastEditors: 赵峰
 * @Descripttion: 设备任务详情--试验数据组件
 * @FilePath: \hifar-platform-client\src\views\hifar\hifar-environmental-test\task\modules\components\detail\TestTaskData.vue
-->
<template>
  <div style="height: 100%; overflow: auto">
    <h-card title="曲线/图谱图片" fixed style="height: auto">
      <h-form
        v-if="imglength"
        style="width: 100%"
        v-model="model_img"
        ref="imageForm"
        :column="1"
        :formData="imageData"
      />
      <div style="display: flex; align-items: center; justify-content: center; width: 100%">
        <a-empty v-if="imglength == 0" />
      </div>
    </h-card>
<!--    <h-card title="曲线" fixed style="height: auto">-->
<!--      <h-form style="width: 100%" v-model="model_cure" ref="curveForm" :column="1" :formData="curveData" />-->
<!--    </h-card>-->
<!--    <h-card title="图谱" fixed style="height: auto">-->
<!--      <h-form style="width: 100%" v-model="model_atlas" ref="atlasForm" :column="1" :formData="atlasData" />-->
<!--    </h-card>-->
    <h-card title="附件" fixed style="height: auto">
      <h-form style="width: 100%" v-model="model_attach" ref="attachForm" :column="1" :formData="attachData" />
    </h-card>
    <h-card title="视频" fixed style="height: auto">
      <h-form style="width: 100%" v-model="model_video" ref="videoForm" :column="1" :formData="videoData" />
    </h-card>
  </div>
</template>

<script>
import { postAction } from '@/api/manage'
export default {
  name: 'testTaskData',
  props: ['testId'],
  data() {
    return {
      model_cure: {},
      model_img: {},
      model_atlas: {},
      model_attach: {},
      model_video: {},
      imglength:0,
      url: {
        attachList: '/MinioBusiness/listByRefId',
        delete: '/MinioBusiness/logicRemoveById',
      },
      imageData: [
        {
          title: '图片',
          key: 'attachIds',
          span: 1,
          component: (
            <h-upload-img
              multiple={true}
              max={100}
              isEdit={false}
              customParams={{ refType: 'test_picture', refId: this.testId }}
              accept="image/png,image/gif,image/jpg,image/jpeg"
              v-decorator={['attachIds', { initialValue: [] }]}
              on-delete={this.handleDelete}
            />
          ),
        },
      ],
      curveData: [
        {
          title: '曲线',
          key: 'attachIds',
          span: 1,
          component: (
            <h-upload-file
              v-decorator={['attachIds', { initialValue: [] }]}
              isEdit={false}
              customParams={{ refType: 'test_cure', refId: this.testId }}
              on-delete={this.handleDelete}
            />
          ),
        },
      ],
      atlasData: [
        {
          title: '图谱',
          key: 'attachIds',
          span: 1,
          component: (
            <h-upload-file
              v-decorator={['attachIds', { initialValue: [] }]}
              isEdit={false}
              customParams={{ refType: 'test_atlas', refId: this.testId }}
              on-delete={this.handleDelete}
            />
          ),
        },
      ],
      attachData: [
        {
          title: '附件',
          key: 'attachIds',
          span: 1,
          component: (
            <h-upload-file
              v-decorator={['attachIds', { initialValue: [] }]}
              isEdit={false}
              customParams={{ refType: 'test_attach', refId: this.testId }}
              on-delete={this.handleDelete}
            />
          ),
        },
      ],
      videoData: [
        {
          title: '视频',
          key: 'attachIds',
          span: 1,
          component: (
            <h-upload-file
              v-decorator={['attachIds', { initialValue: [] }]}
              isEdit={false}
              customParams={{ refType: 'test_video', refId: this.testId }}
              on-delete={this.handleDelete}
            />
          ),
        },
      ],
    }
  },
  created() {
    this.loadData()
  },
  methods: {
    loadData() {
      this.loadImgData()
      this.loadCureData()
      this.loadAtlasData()
      this.loadAttachData()
      this.loadVideoData()
    },
    // 图片
    loadImgData() {
      postAction(this.url.attachList, { refType: 'test_picture', refId: this.testId }).then((res) => {
        if (res.code == 200) {
          const { data } = res
          let fileArr = []
          let obj = {}
          if (data && data.length > 0) {
            data.forEach((item) => {
              fileArr.push({
                fileId: item.id,
                size: item.fileSize,
                status: item.status == 9 ? 'success' : 'exception',
                url: item.filePath,
                name: item.fileName,
                uuid: item.id,
                percent: 100,
                uploadTime: item.createTime,
                secretLevel: item.secretLevel,
                type: item.viewType == 2 ? 'image/jpeg' : 'text/plain',
              })
            })
          }
          obj.attachIds = fileArr
          this.imglength = fileArr.length
          this.model_img = obj
        }
      })
    },
    // 曲线
    loadCureData() {
      postAction(this.url.attachList, { refType: 'test_cure', refId: this.testId }).then((res) => {
        if (res.code == 200) {
          const { data } = res
          let fileArr = []
          let obj = {}
          if (data && data.length > 0) {
            data.forEach((item) => {
              fileArr.push({
                fileId: item.id,
                size: item.fileSize,
                status: item.status == 9 ? 'success' : 'exception',
                url: item.filePath,
                name: item.fileName,
                uuid: item.id,
                percent: 100,
                uploadTime: item.createTime,
                secretLevel: item.secretLevel,
                type: item.viewType == 2 ? 'image/jpeg' : 'text/plain',
              })
            })
          }
          obj.attachIds = fileArr
          this.model_cure = obj
        }
      })
    },
    // 图谱
    loadAtlasData() {
      postAction(this.url.attachList, { refType: 'test_atlas', refId: this.testId }).then((res) => {
        if (res.code == 200) {
          const { data } = res
          let fileArr = []
          let obj = {}
          if (data && data.length > 0) {
            data.forEach((item) => {
              fileArr.push({
                fileId: item.id,
                size: item.fileSize,
                status: item.status == 9 ? 'success' : 'exception',
                url: item.filePath,
                name: item.fileName,
                uuid: item.id,
                percent: 100,
                uploadTime: item.createTime,
                secretLevel: item.secretLevel,
                type: item.viewType == 2 ? 'image/jpeg' : 'text/plain',
              })
            })
          }
          obj.attachIds = fileArr
          this.model_atlas = obj
        }
      })
    },
    // 附件
    loadAttachData() {
      postAction(this.url.attachList, { refType: 'test_attach', refId: this.testId }).then((res) => {
        if (res.code == 200) {
          const { data } = res
          let fileArr = []
          let obj = {}
          if (data && data.length > 0) {
            data.forEach((item) => {
              fileArr.push({
                fileId: item.id,
                size: item.fileSize,
                status: item.status == 9 ? 'success' : 'exception',
                url: item.filePath,
                name: item.fileName,
                uuid: item.id,
                percent: 100,
                uploadTime: item.createTime,
                secretLevel: item.secretLevel,
                type: item.viewType == 2 ? 'image/jpeg' : 'text/plain',
              })
            })
          }
          obj.attachIds = fileArr
          this.model_attach = obj
        }
      })
    },
    // 视频
    loadVideoData() {
      postAction(this.url.attachList, { refType: 'test_video', refId: this.testId }).then((res) => {
        if (res.code == 200) {
          const { data } = res
          let fileArr = []
          let obj = {}
          if (data && data.length > 0) {
            data.forEach((item) => {
              fileArr.push({
                fileId: item.id,
                size: item.fileSize,
                status: item.status == 9 ? 'success' : 'exception',
                url: item.filePath,
                name: item.fileName,
                uuid: item.id,
                percent: 100,
                uploadTime: item.createTime,
                secretLevel: item.secretLevel,
                type: item.viewType == 2 ? 'image/jpeg' : 'text/plain',
              })
            })
          }
          obj.attachIds = fileArr
          this.model_video = obj
        }
      })
    },
    // 图片删除
    handleDelete(file, fileList) {
      postAction(this.url.delete, { id: file.fileId }).then(() => {})
    },
  },
}
</script>