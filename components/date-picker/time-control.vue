<template>
  <div
    ref="wrapper"
    :class="`${prefixCls}__input`"
    tabindex="-1"
    @keydown="handleInput"
  >
    <div
      v-if="enabled.hour"
      :class="[
        `${prefixCls}__unit`,
        visible && unitType === 'hour' ? `${prefixCls}__unit--focused` : ''
      ]"
      @click="handleInputFocus('hour')"
    >
      {{ formattedHour }}
    </div>
    <div v-if="labels.hour" :class="`${prefixCls}__label`">
      {{ labels.hour }}
    </div>
    <template v-if="enabled.minute">
      <div v-if="enabled.hour" :class="`${prefixCls}__separator`">
        {{ separator }}
      </div>
      <div
        :class="[
          `${prefixCls}__unit`,
          visible && unitType === 'minute' ? `${prefixCls}__unit--focused` : ''
        ]"
        @click="handleInputFocus('minute')"
      >
        {{ formattedMinute }}
      </div>
      <div v-if="labels.minute" :class="`${prefixCls}__label`">
        {{ labels.minute }}
      </div>
    </template>
    <template v-if="enabled.second">
      <div v-if="enabled.minute || enabled.hour" :class="`${prefixCls}__separator`">
        {{ separator }}
      </div>
      <div
        :class="[
          `${prefixCls}__unit`,
          visible && unitType === 'second' ? `${prefixCls}__unit--focused` : ''
        ]"
        @click="handleInputFocus('second')"
      >
        {{ formattedSecond }}
      </div>
      <div v-if="labels.second" :class="`${prefixCls}__label`">
        {{ labels.second }}
      </div>
    </template>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue'
import { doubleDigits } from '@vexip-ui/utils'
import { handleKeyEnter } from './helper'

import type { PropType } from 'vue'
import type { TimeType } from './symbol'

export default defineComponent({
  name: 'TimeControl',
  props: {
    unitType: {
      type: String as PropType<TimeType | ''>,
      default: 'date'
    },
    enabled: {
      type: Object as PropType<Record<TimeType, boolean>>,
      default: () => ({})
    },
    activated: {
      type: Object as PropType<Record<TimeType, boolean>>,
      default: () => ({})
    },
    timeValue: {
      type: Object as PropType<Record<TimeType, number>>,
      default: () => ({})
    },
    separator: {
      type: String,
      default: ':'
    },
    visible: {
      type: Boolean,
      default: false
    },
    focused: {
      type: Boolean,
      default: false
    },
    filler: {
      type: String,
      default: '-',
      validator: (value: string) => value.length === 1
    },
    noFiller: {
      type: Boolean,
      default: false
    },
    steps: {
      type: Array as PropType<number[]>,
      default: () => [1, 1, 1]
    },
    ctrlSteps: {
      type: Array as PropType<number[]>,
      default: () => [5, 5, 5]
    },
    labels: {
      type: Object as PropType<Partial<Record<TimeType, string>>>,
      default: () => ({})
    }
  },
  emits: [
    'input',
    'plus',
    'minus',
    'enter',
    'cancel',
    'unit-focus',
    'prev-unit',
    'next-unit'
  ],
  setup(props, { emit }) {
    const prefix = 'vxp-time-picker'

    const wrapper = ref<HTMLElement | null>(null)

    const formattedHour = computed(() => {
      return formatValue('hour')
    })
    const formattedMinute = computed(() => {
      return formatValue('minute')
    })
    const formattedSecond = computed(() => {
      return formatValue('second')
    })

    function formatValue(type: TimeType) {
      return props.noFiller || props.activated[type]
        ? doubleDigits(props.timeValue[type])
        : `${props.filler}${props.filler}`
    }

    function handleInputFocus(type: TimeType) {
      emit('unit-focus', type)
    }

    function handleInput(event: KeyboardEvent) {
      const type = handleKeyEnter(event)

      switch (type) {
        case 'next': {
          emit('next-unit')
          break
        }
        case 'prev': {
          emit('prev-unit')
          break
        }
        case 'up': {
          emit('minus', event.ctrlKey)
          break
        }
        case 'down': {
          emit('plus', event.ctrlKey)
          break
        }
        case 'ok': {
          handleEnter()
          break
        }
        case 'esc': {
          handleCancel()
          break
        }
        default: {
          if (typeof type === 'number') {
            emit('input', type)
          }
        }
      }
    }

    function handleEnter() {
      emit('enter')
    }

    function handleCancel() {
      emit('cancel')
    }

    return {
      prefixCls: prefix,

      formattedHour,
      formattedMinute,
      formattedSecond,

      wrapper,

      handleInputFocus,
      handleInput,

      focus: () => {
        wrapper.value?.focus()
      }
    }
  }
})
</script>
