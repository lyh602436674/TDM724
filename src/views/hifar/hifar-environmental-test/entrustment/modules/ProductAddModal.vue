<!--
 * @Author: 雷宇航
 * @Date: 2022-10-09 10:44:38
 * @fileName: productAddModal.vue
 * @FilePath: tdm724-client\src\views\hifar\hifar-environmental-test\entrustment\modules\productAddModal.vue
 * @Description: 新增样品弹框
-->
<template>
  <h-modal
    :getContainer="getContainer"
    :visible="visible"
    inner
    title="外部新增"
    @cancel="handleCancel"
  >
    <template slot="footer">
      <a-button type="ghost-danger" @click="handleCancel">关闭</a-button>
      <a-button type="primary" @click="handleOk">确定</a-button>
    </template>
    <h-form
      ref="outsideProductForm"
      v-model="model"
      :column="1"
      :formData='formData'
      @change="formChange"
    >
    </h-form>
    <div class="help">样品编号：连续的编号请以' - '分隔，其他情况请以' , '分隔</div>
  </h-modal>
</template>

<script>
export default {
  name: "productAddModal",
  inject: {
    getContainer: {
      default: () => document.body,
    },
  },
  props: {
    entrustType: {
      type: String,
    },
  },
  computed: {
    formData() {
      return [
        {
          title: '样品名称',
          key: 'productName',
          formType: 'input',
        },
        (
          this.entrustType === '1' ? {
            title: '型号/规格',
            key: 'productModel',
            formType: 'input',
          } : {
            title: '图号',
            key: 'productAlias',
            formType: 'input',
          }
        ),
        {
          title: '编号前缀',
          key: 'piecePrefix',
          formType: 'input',
          style: {
            width: '100%',
          }
        },
        {
          title: '编号',
          key: 'pieceNo',
          formType: 'input',
          validate: {
            rules: [{required: true, message: '请输入样品编号',}],
          }
        },
        {
          title: '样品数量',
          key: 'pieceNum',
          formType: 'input-number',
          style: {
            width: '100%',
          }
        },
      ]
    },
  },
  data() {
    return {
      visible: false,
      model: {}
    }
  },
  methods: {
    show() {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.outsideProductForm.form.resetFields()
      })
    },
    formChange(values) {
      this.$emit('callback', values)
      this.handleCancel()
    },
    handleOk() {
      this.$refs.outsideProductForm.validateForm()
    },
    handleCancel() {
      this.visible = false
    },
  }
}
</script>

<style scoped>
.help {
  width: 100%;
  color: red;
  text-align: center;
}
</style>