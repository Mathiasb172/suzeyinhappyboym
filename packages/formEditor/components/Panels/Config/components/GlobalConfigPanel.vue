<script>
import dayjs from 'dayjs'
import hooks from '@ER/hooks'
import DeviceSwitch from '@ER/formEditor/components/DeviceSwitch.vue'
import _ from 'lodash-es'
import { ref, unref, computed, inject } from 'vue'
import { ClickOutside as vClickOutside } from 'element-plus'
import CompleteButton from '@ER/formEditor/components/CompleteButton.vue'
import PanelsConfigComponentsTypeComponent from './TypeComponent.vue'
export default {
  name: 'GlobalConfigPanel',
  inheritAttrs: false,
  customOptions: {}
}
</script>
<script setup>
const {
  target,
  state,
  isPc
} = hooks.useTarget()
const {
  t
} = hooks.useI18n()
const ER = inject('Everright')
const ns = hooks.useNamespace('GlobalConfigPanel')
const compareKeys = ['labelPosition', 'completeButton']
const visible = ref(false)
const buttonRef = ref()
const popoverRef = ref()
const radio1 = ref('pc')
const handleModelValue = (type, value) => {
  const platforms = target.value.isSync ? ['pc', 'mobile'] : [state.platform]
  platforms.forEach((e) => {
    _.set(target.value, `${e}.${type}`, value)
  })
}
const popperPaneRef = computed(() => {
  return _.get(unref(popoverRef), 'popperRef.contentRef', '')
})
let handleConfirm = ''
const handleBeforeChange = () => {
  // visible.value = true
  return new Promise((resolve, reject) => {
    const pcObj = _.pick(unref(target).pc, compareKeys)
    const mobileObj = _.pick(unref(target).mobile, compareKeys)
    if (_.isEqual(pcObj, mobileObj)) {
      resolve(true)
    } else {
      visible.value = true
      handleConfirm = resolve
    }
  })
}
const onClickOutside = () => {
  visible.value = false
}
const handleClick = (type) => {
  visible.value = false
  switch (type) {
    case 2:
      const targetObj = target.value[radio1.value === 'pc' ? 'mobile' : 'pc']
      const sourceObj = _.pick(unref(target)[radio1.value], compareKeys)
      Object.assign(targetObj, _.cloneDeep(sourceObj))
      handleConfirm(true)
      break
  }
}
const options0 = computed(() => {
  return [
    {
      label: t('er.config.globalConfig.labelPosition.top'),
      value: 'top',
      icon: 'labelStructureP1'
    },
    {
      label: t('er.config.globalConfig.labelPosition.left'),
      value: 'left',
      icon: 'labelStructureP2'
    },
    {
      label: t('er.config.globalConfig.labelPosition.right'),
      value: 'right',
      icon: 'labelStructureP3'
    }
  ]
})

//   <el-radio-button label="large">{{t('er.config.globalConfig.componentSize.large')}}</el-radio-button>
// <el-radio-button label="default" >{{t('er.config.globalConfig.componentSize.default')}}</el-radio-button>
// <el-radio-button label="small" >{{t('er.config.globalConfig.componentSize.small')}}</el-radio-button>
const options1 = computed(() => {
  return [
    {
      label: t('er.config.globalConfig.componentSize.large'),
      value: 'large'
    },
    {
      label: t('er.config.globalConfig.componentSize.default'),
      value: 'default'
    },
    {
      label: t('er.config.globalConfig.componentSize.small'),
      value: 'small'
    }
  ]
})
const handleTypeListener = ({ property, data }) => {
  switch (property) {
    case 'labelPosition':
      handleModelValue('labelPosition', data.value)
      break
    case 'size':
      target.value[state.platform].size = data.value
      break
  }
}
</script>
<template>
  <div>
    <el-popover
      virtual-triggering
      :visible="visible"
      ref="popoverRef"
      :virtual-ref="buttonRef"
      :width="200">
      <div>
        <div :class="[ns.e('syncContent')]">
          <el-icon color="#f90">
            <QuestionFilled />
          </el-icon>
          {{ t('er.config.globalConfig.sync.warning') }}
        </div>
        <el-radio-group :class="[ns.e('syncType')]" v-model="radio1" class="ml-4">
          <el-radio label="pc">pc</el-radio>
          <el-radio label="mobile">mobile</el-radio>
        </el-radio-group>
      </div>
      <div :class="[ns.e('syncActions')]">
        <el-button
          size="small"
          :text="true"
          @click="handleClick(1)"
        >{{t('er.public.cancel')}}</el-button>
        <el-button
          size="small"
          type="primary"
          @click="handleClick(2)"
        >
          {{ t('er.public.confirm') }}
        </el-button>
      </div>
    </el-popover>
<!--    <DeviceSwitch justify-content="center"></DeviceSwitch>-->
    <el-form-item :label="t('er.config.globalConfig.sync.label')" label-position="left">
      <el-switch
        ref="buttonRef"
        v-click-outside:[popperPaneRef]="onClickOutside"
        :before-change="handleBeforeChange"
        v-model="target.isSync"/>
    </el-form-item>
    <PanelsConfigComponentsTypeComponent
      v-if="isPc"
      @listener="handleTypeListener"
      property="size"
      :layoutType="2"
      :label="t('er.config.globalConfig.componentSize.label')"
      :val="target[state.platform].size"
      :nodes="options1"
    />
    <PanelsConfigComponentsTypeComponent
      @listener="handleTypeListener"
      property="labelPosition"
      :label="t('er.config.globalConfig.labelPosition.label')"
      :height="66"
      :fontSize="80"
      :val="target[state.platform].labelPosition"
      :nodes="options0"
    />
    <el-form-item v-if="ER.props.isShowCompleteButton" :label="t('er.public.button')">
      <div style="width: 100%;">
        <div>
          <CompleteButton mode="preview"/>
        </div>
        <div>
          <el-row :gutter="8">
            <el-col>
              <el-form-item :label="t('er.public.text')">
                <el-input
                  :model-value="target[state.platform].completeButton.text"
                  show-word-limit
                  :maxlength="20"
                  @update:modelValue="(e) => handleModelValue('completeButton.text', e)"
                ></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="8" style="margin-top: 20px;">
            <el-col :span="12">
              <el-form-item :label="t('er.public.color')">
                <el-color-picker
                  :popper-class="ns.e('completeButtonColor')"
                  :model-value="target[state.platform].completeButton.color"
                  @update:modelValue="(e) => handleModelValue('completeButton.color', e)"
                  show-alpha
                />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item :label="t('er.public.backgroundColor')">
                <el-color-picker
                  :popper-class="ns.e('completeButtonColor')"
                  :model-value="target[state.platform].completeButton.backgroundColor"
                  @update:modelValue="(e) => handleModelValue('completeButton.backgroundColor', e)"
                  show-alpha
                />
              </el-form-item>
            </el-col>
          </el-row>
        </div>
      </div>
    </el-form-item>
  </div>
</template>
