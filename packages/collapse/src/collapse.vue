<template>
  <div class="el-collapse" role="tablist" aria-multiselectable="true">
    <slot></slot>
  </div>
</template>
<script lang='ts'>
import { defineComponent, ref, watch, provide, PropType, Ref, onUnmounted } from 'vue'
import mitt from 'mitt'

export interface CollapseProvider {
  activeNames: Ref
  collapseMitt: mitt.Emitter
}

export default defineComponent({
  name: 'ElCollapse',
  props: {
    accordion: Boolean,
    modelValue: {
      type: [Array, String, Number] as PropType<string | number | Array<string|number>>,
      default: () => [],
    },
  },
  emits: ['update:modelValue'],
  setup(props, { emit }) {
    const activeNames = ref([].concat(props.modelValue))
    const collapseMitt: mitt.Emitter = mitt()

    const setActiveNames = (_activeNames) => {
      activeNames.value = [].concat(_activeNames)
      const value = props.accordion ? activeNames.value[0] : activeNames.value
      emit('update:modelValue', value)
    }

    const handleItemClick = (name) => {
      if(props.accordion) {
        setActiveNames(
          (activeNames.value[0] || activeNames.value[0] === 0) &&
          activeNames.value[0] === name ? '' : name,
        )
      } else {
        let _activeNames = activeNames.value.slice(0)
        const index = _activeNames.indexOf(name)

        if(index > -1) {
          _activeNames.splice(index, 1)
        } else {
          _activeNames.push(name)
        }
        setActiveNames(_activeNames)
      }
    }

    watch(() => props.modelValue, () => {
      activeNames.value = [].concat(props.modelValue)
    })

    collapseMitt.on('item-click', handleItemClick)

    onUnmounted(() => {
      collapseMitt.all.clear()
    })

    provide('collapse', {
      activeNames,
      collapseMitt,
    })

    return {
      activeNames,
      setActiveNames,
      handleItemClick,
    }
  },
})
</script>
