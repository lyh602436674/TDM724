<!--
 * @Author: 雷宇航
 * @Date: 2022-06-16 15:36:11
 * @fileName: NewEntrustmentModal.vue
 * @FilePath: \tdm200-client\tdm200-client\src\views\hifar\hifar-environmental-test\entrustment\modules\NewTestEntrustmentModal.vue
 * @Description: 新版委托单（由之前的三个步骤填写改为一个页面）
-->
<template>
  <h-modal
    :getContainer='getContainer'
    :visible='visible'
    destroyOnClose
    fullScreen inner
    @cancel='handleCancel'>
    <div slot='footer' class='footer'>
      <a-button :loading="submitLoading" style='margin-right: 8px' type='ghost-danger' @click='handleCancel'> 关闭
      </a-button>
      <a-button :loading="submitLoading" style='margin-right: 8px' type='ghost-success' @click='handleTransientSubmit'>
        暂存
      </a-button>
      <a-button type='primary' @click='handleSubmit'>提交</a-button>
    </div>
    <h-card bordered>
      <template slot='title'> {{ handleType === 'add' ? '新增' : '编辑' }}委托试验</template>
      <a-spin :spinning="submitLoading">
        <div class="item-wrapper">
          <div class="item-wrapper-title">
            <span class="title">委托信息</span>
            <span class="description">填写申请单基本信息</span>
          </div>
          <div class="item-wrapper-content">
            <h-form
              ref='entrustFrom'
              v-model='entrustModel'
              :column='3'
              :formData='entrustFormData'
              style='margin-top: 40px'
            >
            </h-form>
          </div>
        </div>
        <div class="item-wrapper">
          <div class="item-wrapper-title">
            <span class="title">试品信息</span>
            <span class="description">选择并填写试品信息</span>
          </div>
          <div class="item-wrapper-content">
            <div style="margin-top:20px">
              <template v-if="tableData.length === 0">
                <a-button
                  icon='plus'
                  size='small'
                  style='margin-right: 10px'
                  type='ghost-primary'
                  @click='selectProductHandle'
                >
                  选择试品
                </a-button>
                <a-button
                  icon='plus'
                  size='small'
                  style='margin-right: 10px'
                  type='ghost-primary'
                  @click='handleManuallyAdd'
                >
                  内部新增
                </a-button>
              </template>
              <a-button v-if='selectedRowKeys.length' icon='minus' size='small' type='danger' @click='handleDelete'> 删除
              </a-button>
            </div>
            <div class="vxe-table-box" style="margin-bottom: 20px">
              <vxe-table
                ref='pieceTable'
                :checkbox-config='{ highlight: true, strict: true }'
                :data='tableData'
                :edit-config="{
                  trigger: 'click',
                  mode: 'cell',
                  activeMethod: activeCellMethod
                }"
                :edit-rules='validRules'
                :valid-config='{ showMessage: false }'
                keep-source
                max-height='600'
                show-overflow
                style='margin-top: 10px'
                @checkbox-all='selectAllEvent'
                @checkbox-change='onSelectChange'
              >
                <vxe-table-column type='checkbox' width='60'></vxe-table-column>
                <vxe-table-column type='seq' width='60'></vxe-table-column>
                <vxe-table-column
                  :edit-render="{
                  name: 'input',
                  attrs: { type: 'text', placeholder: '请输入试品名称' },
                  events:{
                      blur: this.pieceItemChange,
                    }
                }"
                  field='productName'
                  title="试品名称"
                />
                <vxe-table-column
                  :edit-render="{
                   name:'input',
                   attrs: { type: 'text', placeholder: '请输入试品工号' },
                   events:{
                      blur: this.pieceItemChange,
                    }
                }"
                  field='productCode'
                  title="试品工号"
                />
                <vxe-table-column
                  :edit-render="{
                   name:'input',
                   attrs: { type: 'text', placeholder: '请输入试品代号' },
                   events:{
                      blur: this.pieceItemChange,
                    }
                }"
                  field='productAlias'
                  title="试品代号"
                />
                <vxe-table-column
                  :edit-render="{
                    showAsterisk:true,
                    name: 'input',
                    attrs: { type: 'text', placeholder: '请输入试品编号' },
                    events:{
                      blur: this.pieceNoBlur,
                    }
                  }"
                  field='pieceNo'
                  title="试品编号"
                />
              </vxe-table>
            </div>
          </div>
        </div>
        <div class="item-wrapper">
          <div class="item-wrapper-title">
            <span class="title">项目信息</span>
            <span class="description">填写项目基本信息</span>
          </div>
          <div class="item-wrapper-content">
            <a-button
              icon='plus'
              size='small'
              style='margin:20px 0 10px'
              type='ghost-primary'
              @click='handleAddProject'
            >
              项目添加
            </a-button>
            <new-test-project-form ref='ProjectForm' :entrustType="entrustType" :formInfoData='projectInfoData'
                                   :pieceTableData="pieceTableData" style="margin-bottom:20px"
                                   @change='projectFormChange'
                                   @emptyData="emptyDatCallback"></new-test-project-form>
          </div>
        </div>
      </a-spin>
    </h-card>
    <project-add-modal ref='projectAddModal' @change='projectModalCallback'></project-add-modal>
    <product-select-modal ref='ProductSelectModal' @change='productSelectModalCallback'></product-select-modal>
  </h-modal>
</template>

<script>
import NewTestProjectForm from "@views/hifar/hifar-environmental-test/entrustment/components/NewTestProjectForm";
import ProjectAddModal from "@views/hifar/hifar-environmental-test/entrustment/modules/ProjectAddModal";
import ProductSelectModal from "@views/hifar/hifar-environmental-test/entrustment/modules/ProductSelectModal";
import moment from "moment";
import PhemismCustomSelect from "@views/components/PhhemismCustomSelect";
import store from '@/store'
import {cloneDeep, isArray} from 'lodash'
import {postAction} from "@api/manage";
import {randomUUID} from "@/utils/util";
import OutsideProductAddModal from "@views/hifar/hifar-environmental-test/entrustment/modules/OutsideProductAddModal";

export default {
  name: "NewEntrustmentModal",
  components: {OutsideProductAddModal, ProductSelectModal, ProjectAddModal, NewTestProjectForm, PhemismCustomSelect},
  inject: {
    getContainer: {
      default: () => document.body
    }
  },
  data() {
    const nameValid = ({cellValue}) => {
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          if (!cellValue) {
            reject(new Error('试品编号不能为空'))
          } else {
            resolve()
          }
        }, 100)
      })
    }
    return {
      moment,
      visible: false,
      submitLoading: false,
      handleType: 'add',
      submitStatus: 1,
      entrustModel: {},
      entrustType: '',
      tableData: [],
      selectedRowKeys: [],
      projectInfoData: [],
      validRules: {
        pieceNo: [{required: true, message: '试品编号不能为空'}, {validator: nameValid}]
      },
      entrustModelInfo: {},
      pieceModelInfo: [],
      projectModelInfo: [],
      secretLevelArr: [],
      staticTableData: [],
      url: {
        save: "HfEnvEntrustBusiness/saveEntrust",
        setSecretLevel: '/MinioBusiness/modifyAttachSecretLevelByIds',
        edit: "/HfEnvEntrustBusiness/queryById"
      },
    }
  },
  computed: {
    pieceTableData() {
      return this.tableData
    },
    entrustFormData() {
      return [
        {
          key: 'id',
          formType: 'input',
          hidden: true
        },
        {
          title: '委托单',
          key: 'entrustCode',
          formType: 'input',
          disabled: true,
          hidden: this.handleType !== 'edit'
        },
        {
          title: '委托日期',
          key: 'entrustTime',
          formType: 'datePick',
          validate: {
            rules: [{required: true, message: '请选择委托日期'}]
          }
        },
        {
          title: '委托单类型',
          key: 'entrustType',
          formType: 'dict',
          dictCode: 'hf_entrustlist_entrustType',
          disabled: true,
          validate: {
            rules: [{required: true, message: '请选择委托单类型', trigger: 'blur'}]
          },
        },
        {
          title: '送试单位',
          key: 'custName',
          formType: 'input',
          validate: {
            rules: [{required: true, message: '请输入送试单位'}]
          }
        },
        {
          title: '联系人',
          key: 'linkName',
          formType: 'input',
          validate: {
            rules: [{required: false, message: '请输入联系人'}]
          }
        },
        {
          title: '联系方式',
          key: 'linkMobile',
          formType: 'input',
          validate: {
            rules: [{required: false, message: '请输入联系方式'}]
          }
        },
        {
          title: '试验性质',
          key: 'testPropertyCode',
          formType: 'dict',
          dictCode: 'env_test_quality',
          validate: {
            rules: [{required: true, message: '请选择试验性质'}]
          },
          change: (val, option) => {
            this.$refs.entrustFrom.form.setFieldsValue({testPropertyName: option.title})
          }
        },
        {
          title: '',
          key: 'testPropertyName',
          formType: 'input',
          hidden: true
        },
        {
          title: '密级',
          key: 'secretLevelCode',
          formType: 'dict',
          dictCode: 'hf_entrustlist_secondary',
          validate: {
            rules: [{required: false, message: '请选择密级'}]
          },
        },
        {
          title: '是否外包',
          key: 'isExternalManage',
          formType: 'radio',
          radioType: 'radioButton',
          initialValue: 0,
          hidden: true,
          options: [
            {title: '是', value: '1', key: '1', disabled: true},
            {title: '否', value: '0', key: '0'}
          ],
          validate: {
            rules: [{required: false, message: '请选择是否外包'}]
          }
        },
        {
          title: '是否出报告',
          key: 'isReport',
          formType: 'radio',
          radioType: 'radioButton',
          options: [
            {title: '是', value: 1, key: 1},
            {title: '否', value: 2, key: 2}
          ],
          disabled: true,
          validate: {
            rules: [{required: true, message: '请选择是否出报告'}]
          }
        },
        {
          title: '任务编码',
          key: 'outSourceCode',
          formType: 'input',
          validate: {
            rules: [{required: false, message: '请输入任务编码'}]
          }
        },
        {
          title: '委托方批准人',
          key: 'entrustApproval',
          formType: 'input',
          validate: {
            rules: [{required: true, message: '请输入委托方批准人'}]
          }
        },
        {
          title: '备注',
          key: 'remarks',
          formType: 'textarea',
          style: {
            width: '100%',
          },
          autoSize: {minRows: 4},
          span: 3
        },
        {
          title: '委托单附件',
          key: 'attachIds',
          span: 3,
          component: (
            <h-upload-file showSecret={true} secretLevel={1} v-decorator={['attachIds', {initialValue: []}]}/>
          )
        }
      ]
    }
  },
  methods: {
    show(record, type) {
      this.visible = true
      this.handleType = type
      if (record.id) {
        this.handleEdit(record.id)
      } else {
        this.handleAdd(data)
      }
    },
    // 新增默认值
    handleAdd() {
      let userInfo = store.getters.userInfo
      this.entrustModel = {
        linkName: userInfo.idName,
        linkMobile: userInfo.mobile,
        isReport: 2,
        isExternalManage: '0',
        entrustTime: moment(),
        attachIds: [],
        entrustType: '1',
        secretLevelCode: 1,
        custName: userInfo.deptName,
        custId: userInfo.deptId
      }
      this.tableData = []
      this.projectInfoData = []
    },
    handleEdit(id) {
      this.submitLoading = true
      postAction(this.url.edit, {id}).then(res => {
        if (res.code === 200) {
          let obj = Object.assign({}, res.data)
          let fileList = []
          obj.attachInfo.length && obj.attachInfo.forEach(item => {
            fileList.push({
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
              replaceStatus: item.replaceStatus
            })
          })
          obj.attachIds = fileList || []
          obj.entrustTime = obj.entrustTime && obj.entrustTime != 0 ? moment(parseFloat(obj.entrustTime)) : moment()
          this.entrustType = obj.entrustType
          this.entrustModel = obj
          this.tableData = obj.pieceInfo
          this.projectInfoData = obj.projectInfo
        }
      }).finally(() => {
        this.submitLoading = false
      })
    },
    pieceItemChange({row, rowIndex}) {
      function deleteKey(obj, key) {
        let currentObj = cloneDeep(obj)
        delete currentObj[key]
        return currentObj
      }

      setTimeout(() => {
        if (JSON.stringify(deleteKey(row, 'productId')) !== JSON.stringify(deleteKey(this.staticTableData[rowIndex], 'productId'))) {
          row.productId = ''
          row.type = 'inside'
        } else {
          row.productId = this.staticTableData[rowIndex].productId
          row.type = 'selected'
        }
      }, 1)
    },
    // 选择试品
    selectProductHandle() {
      this.$refs.ProductSelectModal.show()
    },
    //选择试品返回数据
    async productSelectModalCallback(val) {
      let tableData = [{
        projectUseFlag: false,
        productId: val.productId,
        productName: val.productName,
        productCode: val.productCode,
        productAlias: val.productAlias,
        pieceNo: val.pieceNo,
        id: randomUUID(),
        type: 'selected'
      }]
      const $table = this.$refs.pieceTable
      await $table.insertAt(tableData, -1)
      const {insertRecords} = $table.getRecordset()
      this.insertRecords = insertRecords
      this.tableData = this.tableData.concat(insertRecords)
      this.staticTableData = cloneDeep(this.tableData)
      this.setProjectPieceNos()
    },
    //内部试品手动新增
    handleManuallyAdd() {
      let product = {
        projectUseFlag: false,
        productId: '',
        productName: '',
        productCode: '',
        productAlias: '',
        pieceNo: '',
        id: randomUUID(),
        type: 'inside'
      }
      this.tableData.push(product)
      this.setProjectPieceNos()
    },
    pieceNoBlur({row}) {
      this.setProjectPieceNos()
    },
    // 试品删除
    handleDelete() {
      const $table = this.$refs.pieceTable
      $table.removeCheckboxRow()
      const {removeRecords} = $table.getRecordset()
      let tableData = this.tableData
      if (tableData.length > 0) {
        this.tableData = tableData.filter((items) => {
          if (!removeRecords.includes(items)) return items
        })
      }
      this.selectedRowKeys = []
      this.setProjectPieceNos()
    },
    // 动态设置项目中已选试品
    setProjectPieceNos() {
      if (this.projectInfoData.length) {
        setTimeout(() => {
          let ProjectForm = this.$refs.ProjectForm
          let projectFormItem = ProjectForm.$refs.projectFormItem
          let tableData = this.$refs.pieceTable.getData()
          let pieceIds = tableData.map(record => record.id).toString()
          let pieceNos = tableData.map(record => record.pieceNo).toString()
          for (let i = 0; i < this.projectInfoData.length; i++) {
            projectFormItem[i].$refs['projectInfoForm' + i].form.setFieldsValue({pieceIds, pieceNos})
            projectFormItem[i].model.pieceNos = pieceNos
          }
        }, 1)
      }
    },
    activeCellMethod({row, column}) {
      // return row.type === 'inside' || (row.type === 'selected' && column.property === 'pieceNo')
      return row.type === 'inside' || row.type === 'selected'
    },
    //  多选
    onSelectChange(records) {
      this.selectedRowKeys = records.records
    },
    // 全选
    selectAllEvent(records) {
      this.selectedRowKeys = records.records
    },
    // 选择项目
    async handleAddProject() {
      let errMap = await this.$refs.pieceTable.validate().catch(errMap => errMap)
      if (errMap) return this.$message.warning('请填写试品编号')
      if (!this.tableData.length) return this.$message.warning('请先添加试品')
      this.$refs.projectAddModal.visible = true
      this.$refs.projectAddModal.getProjectTree()
      this.$refs.projectAddModal.selectedRowKeys = this.projectInfoData && this.projectInfoData.length && this.projectInfoData.map(item => item.id) || []
    },
    // 选择项目弹框返回数据
    projectModalCallback(recordId, record) {
      let tableData = this.$refs.pieceTable.getData()
      let extendRecord = cloneDeep(record)
      if (this.projectInfoData.length) {
        for (let i = 0; i < extendRecord.length; i++) {
          for (let j = 0; j < this.projectInfoData.length; j++) {
            if (extendRecord[i].id === this.projectInfoData[j].id) {
              extendRecord.splice(i, 1)
              i--
              break
            }
          }
        }
      }
      this.projectInfoData = this.projectInfoData.concat(extendRecord.map(item => {
        return {
          ...item,
          needProcess: 2,
          craftUnitId: 0,
          separateMark: 1,
          pieceIds: tableData.map(item => item.id).toString(),
          pieceNos: tableData.map(item => item.pieceNo).toString()
        }
      }))
    },
    //项目信息为空时
    emptyDatCallback() {
      this.submitLoading = false
    },
    // 项目信息返回数据
    projectFormChange(data, bool, status) {
      //bool 是否进行了数据必填验证
      let projectData = cloneDeep(data)
      projectData.forEach((item) => {
        item.pieceIds = isArray(item.pieceIds) && item.pieceIds.length > 0 ? item.pieceIds.join(',') : item.pieceIds
        item.pieceNos = isArray(item.pieceNos) && item.pieceNos.length > 0 ? item.pieceNos.join(',') : item.pieceNos
        item.standardId =
          isArray(item.standardId) && item.standardId.length > 0 ? item.standardId[0] : item.standardId
      })
      if (bool) {
        if (projectData.length) {
          this.projectModelInfo = projectData
        } else {
          this.submitLoading = false
          this.projectModelInfo = []
        }
      } else {
        this.projectModelInfo = projectData
      }
      this.setSecretLevelByFileList(this.secretLevelArr)
      this.submitRequest(this.submitStatus)
    },
    //暂存 不需要验证任何表单和表格
    handleTransientSubmit() {
      this.submitLoading = true
      this.submitStatus = 1
      this.validateEntrustForm(false).then(res => {
        let entrustModelInfo = cloneDeep(res)
        this.buildEntrustData(entrustModelInfo)
        this.getPiecesTableData(false).then(data => {
          this.pieceModelInfo = data
          this.$refs.ProjectForm.submitHandle(false)
        })
      })
    },
    //提交
    handleSubmit() {
      this.submitLoading = true
      this.submitStatus = 10
      this.validateEntrustForm(true).then(res => {
        let entrustModelInfo = cloneDeep(res)
        this.buildEntrustData(entrustModelInfo)
        this.getPiecesTableData(true).then(data => {
          if (data.length) {
            this.pieceModelInfo = data
            this.$refs.ProjectForm.submitHandle(true, 10)
          } else {
            this.submitLoading = false
            return this.$message.warning('请添加试品！')
          }
        })
      })
    },
    // 暂存-提交请求
    submitRequest(status) {
      let {entrustModelInfo, pieceModelInfo, projectModelInfo} = this
      let params = {entrustModelInfo, pieceModelInfo, projectModelInfo, status,}
      postAction(this.url.save, params).then(res => {
        if (res.code === 200) {
          this.$message.success(status === 1 ? '暂存成功' : "提交成功")
          this.handleCancel()
          this.$emit('change', true)
        } else {
          this.$message.warning(res.msg)
        }
      }).finally(() => {
        this.submitLoading = false
      })
    },
    // 构建委托表单数据
    buildEntrustData(entrustModelInfo) {
      let attachIds = [], secretLevelArr = []
      entrustModelInfo.attachIds.length && entrustModelInfo.attachIds.forEach(item => {
        attachIds.push(item.fileId)
        secretLevelArr.push({
          id: item.fileId,
          secretLevel: item.secretLevel
        })
      })
      this.secretLevelArr = secretLevelArr
      entrustModelInfo.attachIds = attachIds.length > 0 ? attachIds.join(',') : ''
      entrustModelInfo.isReport = entrustModelInfo.isReport ? entrustModelInfo.isReport : 2
      entrustModelInfo.isExternalManage = entrustModelInfo.isExternalManage ? entrustModelInfo.isExternalManage : '0'
      entrustModelInfo.custId = isArray(entrustModelInfo.custId) && entrustModelInfo.custId.length > 0 ? entrustModelInfo.custId[0] : entrustModelInfo.custId
      entrustModelInfo.entrustTime = entrustModelInfo.entrustTime ? entrustModelInfo.entrustTime.valueOf() : ''
      this.entrustModelInfo = entrustModelInfo
    },
    // 给附件设置密级
    setSecretLevelByFileList(secretLevelArr) {
      postAction(this.url.setSecretLevel, {attachInfo: secretLevelArr})
    },
    // 验证委托信息的form表单 获取表单数据
    validateEntrustForm(bool) {
      // bool是否进行表单验证
      let entrustFrom = this.$refs.entrustFrom.form
      return new Promise((resolve, reject) => {
        if (bool) {
          entrustFrom.validateFieldsAndScroll((err, values) => {
            if (!err) {
              resolve(values)
            } else {
              this.submitLoading = false
            }
          })
        } else {
          let values = entrustFrom.getFieldsValue()
          resolve(values)
        }
      })
    },
    // 获取试品表格数据
    getPiecesTableData(bool) {
      // bool 是否进行表格验证
      let pieceTable = this.$refs.pieceTable
      let pieceInfoData = []
      return new Promise((resolve, reject) => {
        if (bool) {
          pieceTable.validate().then(() => {
            let tableData = pieceTable.getData()
            if (tableData.length === 0) {
              pieceInfoData = []
            } else {
              pieceInfoData = tableData
            }
            resolve(pieceInfoData)
          }).catch(err => {
            reject(err)
            this.submitLoading = false
          })
        } else {
          resolve(pieceTable.getData())
        }
      })
    },
    handleCancel() {
      this.visible = false
      this.submitLoading = false
      this.entrustType = '1'
    },
  },
}
</script>

<style lang="less" scoped>
.item-wrapper {
  margin-bottom: 10px;
  border: 1px solid #d9d9d9;

  &-title {
    width: 100%;
    height: 40px;
    border-bottom: 1px solid #d9d9d9;
    padding: 0 20px;
    background-color: #ecf5ff;

    .title {
      color: #000;
      font-size: 18px;
      line-height: 40px;
      position: relative;

      &:before {
        content: "";
        width: 3px;
        height: 40px;
        background-color: #409EFF;
        position: absolute;
        left: -20px;
      }
    }

    .description {
      display: inline-block;
      margin-left: 10px;
    }
  }

  &-content {
    padding: 0 20px;
  }
}

/deep/ .ant-radio-button-wrapper:first-child {
  border-radius: 4px;
}

/deep/ .ant-radio-button-wrapper:last-child {
  border-radius: 4px;
}

/deep/ .ant-radio-button-wrapper {
  border-radius: 4px;
  margin-right: 10px;
  min-width: 80px;
  text-align: center;
}

/deep/ .ant-modal-close-x {
  line-height: 33px !important;
}

/deep/ .vxe-table-box .vxe-table .vxe-body--column.col--valid-error .vxe-cell--valid .vxe-cell--valid-msg {
  background: transparent !important;
}
</style>