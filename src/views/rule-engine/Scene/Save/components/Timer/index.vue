<template>
  <j-form
    ref='timerForm'
    :model='formModel'
    layout='vertical'
    :colon='false'
  >
    <j-form-item name='trigger'>
      <j-radio-group
        v-model:value='formModel.trigger'
        :options='[
          { label: "按周", value: "week" },
          { label: "按月", value: "month" },
          { label: "cron表达式", value: "cron" },
        ]'
        option-type='button'
        button-style='solid'
        @change='triggerChange'
      />
    </j-form-item>
    <j-form-item v-if='showCron' name='cron' :rules="cronRules">
      <j-input placeholder='corn表达式' v-model:value='formModel.cron' @change='updateValue' />
    </j-form-item>
    <template v-else>
      <j-form-item name='when'>
        <WhenOption v-model:value='formModel.when' :type='formModel.trigger' @change='updateValue' />
      </j-form-item>
      <j-form-item name='mod'>
        <j-radio-group
          v-model:value='formModel.mod'
          :options='[
          { label: "周期执行", value: "period" },
          { label: "执行一次", value: "once" },
        ]'
          option-type='button'
          button-style='solid'
          @change='updateValue'
        />
      </j-form-item>
    </template>
    <j-space v-if='showOnce' style='display: flex;gap: 24px'>
      <j-form-item :name="['once', 'time']">
        <j-time-picker
          valueFormat='HH:mm:ss'
          v-model:value='formModel.once.time'
          style='width: 100%'
          format='HH:mm:ss'
          @change='updateValue'
        />
      </j-form-item>
      <j-form-item> 执行一次</j-form-item>
    </j-space>
    <j-space v-if='showPeriod' style='display: flex;gap: 24px'>
      <j-form-item>
        <j-time-range-picker
          valueFormat='HH:mm:ss'
          :value='[
            formModel.period.from,
            formModel.period.to,
           ]'
          @change='(v) => {
                  formModel.period.from = v[0]
                  formModel.period.to = v[1]
                  updateValue()
              }'
        />
      </j-form-item>
      <j-form-item>每</j-form-item>
      <j-form-item
        :name='["period", "every"]'
        :rules='[{ required: true, message: "请输入时间" }]'
      >
        <j-input-number
          placeholder='请输入时间'
          style='max-width: 170px'
          :precision='0'
          :min='1'
          :max='unitMax'
          v-model:value='formModel.period.every'
          @change='updateValue'
        >
          <template #addonAfter>
            <j-select
              v-model:value='formModel.period.unit'
              :options='[
                { label: "秒", value: "seconds" },
                { label: "分", value: "minutes" },
                { label: "小时", value: "hours" },
              ]'
              @select='periodUnitChange'
            />
          </template>
        </j-input-number>
      </j-form-item>
      <j-form-item>执行一次</j-form-item>
    </j-space>
  </j-form>
</template>

<script setup lang='ts' name='Timer'>
import type { PropType } from 'vue'
import dayjs from 'dayjs'
import WhenOption from './WhenOption.vue'
import { cloneDeep } from 'lodash-es'
import type { OperationTimer } from '../../../typings'
import { isCron } from '@/utils/regular'
import { defineExpose } from 'vue'

type NameType = string[] | string

type Emit = {
  (e: 'update:value', data: OperationTimer): void
}

const props = defineProps({
  name: {
    type: [String, Array] as PropType<NameType>,
    default: ''
  },
  value: {
    type: Object,
    default: () => ({})
  }
})

const emit = defineEmits<Emit>()
const unitMax = ref<number>(99)

const cronRules = [
  { max: 64, message: '最多可输入64个字符' },
  {
    validator: async (_: any, v: string) => {
      if (v) {
        if (!isCron(v)) {
          return Promise.reject(new Error('请输入正确的cron表达式'));
        }
      } else {
        return Promise.reject(new Error('请输入cron表达式'));
      }
      return Promise.resolve();
    }
  }
]

const formModel = reactive<OperationTimer>({
  trigger: 'week',
  when: props.value.when || [],
  mod: 'period',
  cron: undefined,
  once: {
    time: dayjs(new Date()).format('HH:mm:ss')
  },
  period: {
    from: dayjs(new Date()).startOf('day').format('HH:mm:ss'),
    to: dayjs(new Date()).endOf('day').format('HH:mm:ss'),
    every: 1,
    unit: 'seconds'
  }
})
const timerForm = ref()

const showCron = computed(() => {
  return formModel.trigger === 'cron'
})

const showOnce = computed(() => {
  return formModel.trigger !== 'cron' && formModel.mod === 'once'
})

const showPeriod = computed(() => {
  return formModel.trigger !== 'cron' && formModel.mod === 'period'
})


const updateValue = () => {

  const cloneValue = cloneDeep(formModel)
  if (cloneValue.trigger === 'cron') {
    delete cloneValue.when
  } else {
    delete cloneValue.cron
  }

  if (cloneValue.mod === 'period') {
    delete cloneValue.once
  } else {
    delete cloneValue.period
  }
  emit('update:value', cloneValue)
}

const triggerChange = () => {
  formModel.when = []
  formModel.cron = undefined
  updateValue()
}

/**
 * 频率单位切换
 * @param v
 */
const periodUnitChange = (v: any) => {
  if(v === 'hours') {
    unitMax.value = 99999
  } else {
    unitMax.value = 99
  }
  formModel.period!.every = 1
  updateValue()
}

defineExpose({
  validateFields: () => new Promise(async (resolve)  => {
    const data = await timerForm.value?.validateFields()
    resolve(data)
  })
})

Object.assign(formModel, props.value)
formModel.when = props.value.when || []

watchEffect(() => {
  if(props.value?.period?.unit === 'hours') {
    unitMax.
      value = 99999
  } else {
    unitMax.value = 99
  }
})


</script>

<style scoped lang='less'>

</style>