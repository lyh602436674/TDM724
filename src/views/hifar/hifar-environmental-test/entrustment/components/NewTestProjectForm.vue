<!--
 * @Author: 雷宇航
 * @Date: 2022-06-22 14:39:44
 * @fileName: NewTestProjectForm.vue
 * @FilePath: tdm200-client\src\views\hifar\hifar-environmental-test\entrustment\components\NewTestProjectForm.vue
 * @Description: 对此文件的描述...
-->
<template>
  <div style="height: 100%">
    <div v-for="(item, index) in formInfoDataList" :key="index" class="panel-custom">
      <div class="panel-custom-item">
        <div class="panel-custom-item-left">
          <new-test-project-form-item ref="projectFormItem" :entrustType="entrustType" :index="index"
                                      :pieceTableData="pieceTableData" :project="item"></new-test-project-form-item>
        </div>
        <div class="panel-custom-item-right">
          <a-button
            icon="delete"
            size="small"
            style="width: 32px; height: 32px; margin-top: 10px"
            type="danger"
            @click="deleteProjectHandle(item, index)"
          ></a-button>
        </div>
      </div>
    </div>
    <div v-if="formInfoDataList.length === 0" class="emptyStyle">
      <a-empty/>
    </div>
    <a-alert v-if="messageErrorShow" :message="messageError" show-icon type="error"/>
  </div>
</template>

<script>
import NewTestProjectFormItem from './NewTestProjectFormItem'
import entrustmentMixins from "@views/hifar/hifar-environmental-test/entrustment/components/entrustmentMixins";

export default {
  mixins: [entrustmentMixins],
  props: {
    formInfoData: {
      type: Array,
      default: () => []
    },
    pieceTableData: {
      type: Array,
      default: () => []
    },
    entrustType: {
      type: [String, Number],
      default: ""
    }
  },
  components: {
    NewTestProjectFormItem
  },
  watch: {
    formInfoData: {
      deep: true,
      immediate: true,
      handler(val) {
        if (!val || !val.length) {
          this.formInfoDataList = []
        } else {
          this.formInfoDataList = val
        }
      }
    }
  },
  data() {
    return {
      form: null,
      messageErrorShow: false,
      messageError: '',
      projectModel: {
        needProcess: 1
      },
      formInfoDataList: [],
      selectRow: null
    }
  },
  methods: {
    // 删除项目
    deleteProjectHandle(item, index) {
      const {formInfoDataList} = this
      formInfoDataList.splice(index, 1)
      this.formInfoDataList = formInfoDataList
    },
    submitHandle(bool) {
      // bool 是否进行验证
      // status 提交/暂存
      const {formInfoDataList} = this
      let projectResult = [];
      if (!formInfoDataList.length) {
        if (bool) {
          this.$emit('emptyData')
          return this.$message.warning('请添加项目')
        } else {
          return this.$emit('change', [], bool)
        }
      } else {
        let errMap = null
        for (let i = 0; i < formInfoDataList.length; i++) {
          let that = this.$refs.projectFormItem[i]
          let projectForm = that.$refs['projectInfoForm' + [i]]
          // 判断是否是 结构化条件 试验项目（根据项目编码判断）
          let filterUnitCodeFlag = this.filterUnitCode(formInfoDataList[i].unitCode)
          if (filterUnitCodeFlag) {
            that.resultEcharts()
          }
          let testConditionDataForm = that.$refs['testConditionDataForm' + [i]]
          let tabItemTableAllData = [];
          if (filterUnitCodeFlag) {
            let testConditionTabItem = that.$refs.testConditionTabItem;
            [].forEach.call(testConditionTabItem, (item, index) => {
              let tabPanelItem = that.model.abilityRequire[index]
              let _item_ = item.$refs['pointTable' + [i] + [index]]
              tabItemTableAllData.push({
                title: tabPanelItem.title,
                type: tabPanelItem.type,
                abilityInfo: _item_.getData()
              })
            })
          } else {
            tabItemTableAllData.push({
              title: '试验条件',
              type: 'default',
              abilityInfo: that.$refs.testConditionTabItem.$refs['pointTable' + [i] + 0].getData()
            })
          }
          if (bool) {
            projectForm.form.validateFieldsAndScroll((error, val) => {
              if (error) {
                errMap = error
              } else {
                testConditionDataForm.form.validateFieldsAndScroll((err, value) => {
                  if (err) {
                    errMap = err
                  } else {
                    let testCondition = value.testCondition
                    let attachIds = value.attachIds.map(item => item.fileId).toString()
                    let projectFormValue = val
                    projectFormValue.testCondition = testCondition;
                    projectFormValue.unitName = that.model.unitName;
                    projectFormValue.predictDuration = that.predictDuration; //预计时长
                    projectFormValue.abilityRequire = tabItemTableAllData;
                    projectFormValue.showAbilityRequireImg = that.temperatureCurveFlag || that.humidityCurveFlag
                    projectFormValue.curveUrl = that.curveUrl;
                    projectFormValue.attachIds = attachIds;
                    projectResult.push(projectFormValue)
                  }
                })
              }
            })
            // 在循环中校验表单得用一个变量记录异常 不然无法跳出循环 并且无法进行表单校验
            if (errMap) return this.$emit('emptyData')
          } else {
            let projectFormValue = projectForm.form.getFieldsValue()
            let testConditionData = testConditionDataForm.form.getFieldsValue()
            projectFormValue.testCondition = testConditionData.testCondition
            projectFormValue.attachIds = testConditionData.attachIds.map(item => item.fileId).toString()
            projectFormValue.unitName = that.model.unitName
            projectFormValue.predictDuration = that.predictDuration; //预计时长
            projectFormValue.abilityRequire = tabItemTableAllData
            projectFormValue.showAbilityRequireImg = that.temperatureCurveFlag || that.humidityCurveFlag
            projectFormValue.curveUrl = that.curveUrl
            projectResult.push(projectFormValue)
          }
        }
        if (errMap) {
          return this.$emit('emptyData')
        } else {
          this.$emit('change', projectResult, bool)
        }
      }
    }
  }
}
</script>
<style lang="less" scoped>
.panel-custom {
  border-radius: 5px;
  margin-bottom: 10px;
  border: 0;
  overflow: hidden;

  .panel-custom-item {
    display: flex;
    margin-bottom: 10px;

    .panel-custom-item-left {
      width: 98%;
      min-height: 226px;
      border: 1px solid #e8e8e8;
      padding: 0 20px 20px;

      .collapseStyle {
        width: 93%;
        margin-left: 6%;
        font-size: 12px;
        margin-bottom: 18px;
      }
    }

    .panel-custom-item-right {
      padding-left: 5px;
      display: flex;
      align-items: center;
      flex-direction: column;
    }
  }
}

.emptyStyle {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  border: 1px solid #ccc;
  padding: 100px 0;
}
</style>