<template>
  <a
    :class="className"
    :href="props.to"
    :target="props.target"
    @click="handleClick"
  >
    <slot name="icon">
      <Icon v-if="props.icon" :class="`${prefix}__icon`" :icon="props.icon"></Icon>
    </slot>
    <slot></slot>
  </a>
</template>

<script lang="ts">
import { defineComponent, computed } from 'vue'
import { Icon } from '@/components/icon'
import { useProps, booleanProp } from '@vexip-ui/config'

import type { PropType } from 'vue'

export type LinkerType = 'default' | 'primary' | 'success' | 'error' | 'warning' | 'info'

const linkerTypes = Object.freeze<LinkerType>(['default', 'primary', 'success', 'error', 'warning', 'info'])

export default defineComponent({
  name: 'Linker',
  components: {
    Icon
  },
  props: {
    to: String,
    type: String as PropType<LinkerType>,
    icon: Object,
    underline: booleanProp,
    disabled: booleanProp,
    target: String
  },
  emits: ['click'],
  setup(_props, { emit }) {
    const props = useProps('linker', _props, {
      to: {
        default: null,
        static: true
      },
      type: {
        default: 'default' as LinkerType,
        validator: (value: LinkerType) => linkerTypes.includes(value)
      },
      icon: null,
      underline: false,
      disabled: false,
      target: '_blank'
    })

    const prefix = 'vxp-linker'

    const className = computed(() => {
      return {
        [prefix]: true,
        [`${prefix}-vars`]: true,
        [`${prefix}--${props.type}`]: props.type !== 'default',
        [`${prefix}--disabled`]: props.disabled,
        [`${prefix}--underline`]: props.underline
      }
    })

    function handleClick(event: MouseEvent) {
      emit('click', event)
    }

    return {
      props,
      prefix,

      className,

      handleClick
    }
  }
})
</script>
