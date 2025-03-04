<template>
  <div :class="className" :style="itemStyle">
    <div :class="`${prefix}__signal`" @click="handleSignalClick">
      <slot name="signal">
        <div :class="`${prefix}__pointer`"></div>
      </slot>
    </div>
    <div :class="`${prefix}__line`" :style="lineStyle"></div>
    <div ref="content" :class="`${prefix}__content`">
      <slot></slot>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, computed, inject, onBeforeUnmount } from 'vue'
import { useProps, booleanProp } from '@vexip-ui/config'
import { TIMELINE_STATE } from './symbol'

import type { PropType } from 'vue'
import type { TimelinkItemType, ItemState } from './symbol'

const timelineItemTypes = Object.freeze<TimelinkItemType>(['default', 'success', 'error', 'warning', 'disabled'])

export default defineComponent({
  name: 'TimelineItem',
  props: {
    type: String as PropType<TimelinkItemType>,
    color: String,
    label: [Number, String],
    dashed: booleanProp,
    lineColor: String,
    spacing: [Number, String]
  },
  emits: ['signal-click'],
  setup(_props, { emit }) {
    const props = useProps('timelineItem', _props, {
      type: {
        default: 'default' as TimelinkItemType,
        validator: (value: TimelinkItemType) => timelineItemTypes.includes(value)
      },
      color: '',
      label: {
        default: null,
        static: true
      },
      dashed: null,
      lineColor: null,
      spacing: null
    })

    const timelineState = inject(TIMELINE_STATE, null)

    const prefix = 'vxp-timeline'
    const currentLabel = ref(props.label)

    const className = computed(() => {
      return {
        [`${prefix}__item`]: true,
        [`${prefix}__item--${props.type}`]: props.type !== 'default'
      }
    })
    const itemStyle = computed(() => {
      const spacing = props.spacing || props.spacing === 0 ? props.spacing : timelineState?.spacing
      const style: Record<string, any> = {
        '--vxp-timeline-item-span': typeof spacing === 'number' ? `${spacing}px` : spacing
      }

      if (props.color) {
        style['--vxp-timeline-pointer-color'] = props.color
        style['--vxp-timeline-pointer-b-color'] = props.color
      }

      return style
    })
    const lineStyle = computed(() => {
      const isDashed = props.dashed ?? timelineState?.dashed ?? false
      const color = props.lineColor ?? timelineState?.lineColor

      return {
        borderLeftStyle: isDashed ? ('dashed' as const) : undefined,
        borderLeftColor: color
      }
    })

    if (timelineState) {
      const state: ItemState = reactive({
        label: currentLabel
      })

      timelineState.increaseItem(state)

      onBeforeUnmount(() => {
        timelineState.decreaseItem(state)
      })
    }

    function handleSignalClick() {
      emit('signal-click', currentLabel.value)
      timelineState?.handleSignalClick(currentLabel.value)
    }

    return {
      prefix,

      className,
      itemStyle,
      lineStyle,

      handleSignalClick
    }
  }
})
</script>
