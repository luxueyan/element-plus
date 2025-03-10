<template>
  <label
    class="el-checkbox-button"
    :class="[
      size ? 'el-checkbox-button--' + size : '',
      { 'is-disabled': isDisabled },
      { 'is-checked': isChecked },
      { 'is-focus': focus },
    ]"
    role="checkbox"
    :aria-checked="isChecked"
    :aria-disabled="isDisabled"
  >
    <input
      v-if="trueLabel || falseLabel"
      v-model="model"
      class="el-checkbox-button__original"
      type="checkbox"
      :name="name"
      :disabled="isDisabled"
      :true-value="trueLabel"
      :false-value="falseLabel"
      @change="handleChange"
      @focus="focus = true"
      @blur="focus = false"
    >
    <input
      v-else
      v-model="model"
      class="el-checkbox-button__original"
      type="checkbox"
      :name="name"
      :disabled="isDisabled"
      :value="label"
      @change="handleChange"
      @focus="focus = true"
      @blur="focus = false"
    >

    <span
      v-if="$slots.default || label"
      class="el-checkbox-button__inner"
      :style="isChecked ? activeStyle : null"
    >
      <slot>{{ label }}</slot>
    </span>

  </label>
</template>
<script lang='ts'>
import {
  defineComponent,
  ref,
  computed,
  // nextTick,
  watch,
} from 'vue'
import { useCheckbox } from './useCheckbox'

export default defineComponent({
  name: 'ElCheckboxButton',
  props: {
    modelValue: {
      type: [Object, Boolean],
      default: () => undefined,
    },
    label: {
      type: [Object, Boolean, String],
      default: () => ({}),
    },
    indeterminate: Boolean,
    disabled: Boolean,
    checked: Boolean,
    name: {
      type: String,
      default: undefined,
    },
    trueLabel: {
      type: [String, Number],
      default: undefined,
    },
    falseLabel: {
      type: [String, Number],
      default: undefined,
    },
  },
  emits: ['update:modelValue', 'change'],
  setup(props, { emit }) {
    const { elForm, isGroup, _checkboxGroup, _elFormItemSize, elFormItem, ELEMENT } = useCheckbox()
    const selfModel = ref(false)
    const focus = ref(false)
    const isLimitExceeded = ref(false)
    const store = computed(() => _checkboxGroup ? _checkboxGroup.modelValue.value : props.modelValue)
    const model = computed({
      get() {
        return isGroup.value ? store.value : props.modelValue !== undefined ? props.modelValue : selfModel
      },

      set(val: any) {
        if (isGroup.value) {
          isLimitExceeded.value = false

          if (_checkboxGroup.min !== undefined && val.length < _checkboxGroup.min) {
            isLimitExceeded.value = true
          }
          if (_checkboxGroup.max !== undefined && val.length > _checkboxGroup.max) {
            isLimitExceeded.value = true
          }

          isLimitExceeded.value === false && _checkboxGroup.changeEvent?.(val)
        } else {
          emit('update:modelValue', val)
          selfModel.value = val
        }
      },
    })
    const isChecked = computed(() => {
      if (Object.prototype.toString.call(model.value) === '[object Boolean]') {
        return model.value
      } else if (Array.isArray(model.value)) {
        return model.value.includes(props.label)
      } else if (model.value !== null && model.value !== undefined) {
        return model.value === props.trueLabel
      }
    })
    const isLimitDisabled = computed(() => {
      const max = _checkboxGroup.max
      const min = _checkboxGroup.min
      return !!(max || min) && (model.value.length >= max && !isChecked.value) ||
          (model.value.length <= min && isChecked.value)
    })
    const isDisabled = computed(() => {
      return isGroup.value
        ? _checkboxGroup.disabled || props.disabled ||  (elForm as any || {} as any).disabled || isLimitDisabled.value
        : props.disabled || (elForm as any || {} as any).disabled
    })

    const activeStyle = computed(() => {
      return {
        backgroundColor: _checkboxGroup.fill || '',
        borderColor: _checkboxGroup.fill || '',
        color: _checkboxGroup.textColor || '',
        'box-shadow': '-1px 0 0 0 ' + _checkboxGroup.fill,
      }
    })

    const size = computed(() => _checkboxGroup.checkboxGroupSize || _elFormItemSize || (ELEMENT || {}).size)

    function addToStore() {
      if (
        Array.isArray(model.value) &&
        !model.value.includes(props.label)
      ) {
        model.value.push(props.label)
      } else {
        model.value = props.trueLabel || true
      }
    }

    function handleChange(e: UIEvent) {
      if (isLimitExceeded.value) return
      let value = ref(undefined)
      if ((e.target as HTMLInputElement).checked) {
        value.value = props.trueLabel === undefined ? true : props.trueLabel
      } else {
        value.value = props.falseLabel === undefined ? false : props.falseLabel
      }

      emit('change', value.value, e)
      /**
       * to discuss it's useful
       */
      // nextTick(() => {
      //   if (isGroup.value) {
      //     _checkboxGroup.changeEvent?.(_checkboxGroup.modelValue.value)
      //   }
      // })
    }

    watch(() => props.modelValue, (val) => {
      elFormItem.changeEvent?.(val)
    })

    props.checked && addToStore()


    return {
      focus,
      isChecked,
      isDisabled,
      model,
      handleChange,
      activeStyle,
      size,
    }
  },
})
</script>
<style scoped>
</style>
