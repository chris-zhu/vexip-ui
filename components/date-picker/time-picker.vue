<template>
  <div
    ref="wrapper"
    :class="className"
    @click="handleTirggerClick"
    @clickoutside="finishInput"
  >
    <div ref="reference" :class="`${prefixCls}__selector`">
      <div v-if="hasPrefix" :class="`${prefixCls}__icon--prefix`" :style="{ color: props.prefixColor }">
        <slot name="prefix">
          <Icon :icon="props.prefix"></Icon>
        </slot>
      </div>
      <div :class="`${prefixCls}__control`">
        <TimeControl
          ref="start"
          :unit-type="currentState === 'start' ? startState.column : ''"
          :enabled="startState.enabled"
          :activated="startState.activated"
          :time-value="startState.timeValue"
          :steps="props.steps"
          :ctrl-steps="props.ctrlSteps"
          :focused="focused"
          :visible="currentVisible"
          :separator="props.separator"
          :filler="props.filler"
          :no-filler="props.noFiller"
          :labels="props.labels"
          @input="handleInput"
          @plus="handlePlus"
          @minus="handleMinus"
          @enter="handleEnter"
          @cancel="handleCancel"
          @unit-focus="handleStartInput"
          @prev-unit="enterColumn('prev')"
          @next-unit="enterColumn('next')"
        ></TimeControl>
        <template v-if="props.isRange">
          <div
            :class="[`${prefixCls}__exchange`, props.exchange ? `${prefixCls}__exchange--enabled` : '']"
            @click="handleExchangeClick"
          >
            <Icon><ArrowRightArrowLeft></ArrowRightArrowLeft></Icon>
          </div>
          <TimeControl
            ref="end"
            :unit-type="currentState === 'end' ? endState.column : ''"
            :enabled="endState.enabled"
            :activated="endState.activated"
            :time-value="endState.timeValue"
            :steps="props.steps"
            :ctrl-steps="props.ctrlSteps"
            :focused="focused"
            :visible="currentVisible"
            :separator="props.separator"
            :filler="props.filler"
            :no-filler="props.noFiller"
            :labels="props.labels"
            @input="handleInput"
            @plus="handlePlus"
            @minus="handleMinus"
            @enter="handleEnter"
            @cancel="handleCancel"
            @unit-focus="handleEndInput"
            @prev-unit="enterColumn('prev')"
            @next-unit="enterColumn('next')"
          ></TimeControl>
        </template>
      </div>
      <transition name="vxp-fade">
        <div
          v-if="!props.disabled && props.clearable && isHover && lastValue"
          :class="`${prefixCls}__clear`"
          @click.stop="handleClear"
        >
          <Icon><CircleXmark></CircleXmark></Icon>
        </div>
        <div v-else :class="`${prefixCls}__icon--suffix`" :style="{ color: props.suffixColor }">
          <slot name="suffix">
            <Icon :icon="props.suffix || ClockR"></Icon>
          </slot>
        </div>
      </transition>
    </div>
    <Portal :to="transferTo">
      <transition :name="props.transitionName">
        <div
          v-show="currentVisible"
          ref="popper"
          :class="[`${prefixCls}__popper`, `${prefixCls}-vars`]"
          @click.stop="handleFocused"
        >
          <div :class="`${prefixCls}__pane`">
            <div v-if="props.shortcuts.length" :class="[`${prefixCls}__list`, `${prefixCls}__list--sub`]">
              <div
                v-for="(item, index) in props.shortcuts"
                :key="index"
                :class="`${prefixCls}__shortcut`"
                :title="item.name"
                @click="handleShortcut(index)"
              >
                {{ item.name }}
              </div>
            </div>
            <div :class="`${prefixCls}__list`">
              <div style="display: flex;">
                <TimeWheel
                  v-model:hour="startState.timeValue.hour"
                  v-model:minute="startState.timeValue.minute"
                  v-model:second="startState.timeValue.second"
                  :no-arrow="props.noArrow"
                  :candidate="props.candidate"
                  :steps="props.steps"
                  :pointer="props.pointer"
                  @change="handleWheelChange"
                  @toggle-col="handleStartInput"
                ></TimeWheel>
                <TimeWheel
                  v-if="isRange"
                  v-model:hour="endState.timeValue.hour"
                  v-model:minute="endState.timeValue.minute"
                  v-model:second="endState.timeValue.second"
                  :no-arrow="props.noArrow"
                  :candidate="props.candidate"
                  :steps="props.steps"
                  :pointer="props.pointer"
                  @change="handleWheelChange"
                  @toggle-col="handleEndInput"
                ></TimeWheel>
              </div>
              <div v-if="!props.noAction" :class="`${prefixCls}__action`">
                <Button text size="small" @click="handleCancel">
                  {{ props.cancelText || locale.cancel }}
                </Button>
                <Button type="primary" size="small" @click="finishInput">
                  {{ props.confirmText || locale.confirm }}
                </Button>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </Portal>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, computed, watch, inject, toRef, nextTick } from 'vue'
import { Button } from '@/components/button'
import { Icon } from '@/components/icon'
import { Portal } from '@/components/portal'
import TimeControl from './time-control.vue'
import TimeWheel from './time-wheel.vue'
import { VALIDATE_FIELD, CLEAR_FIELD } from '@/components/form-item'
import { useHover, usePopper, placementWhileList, useClickOutside } from '@vexip-ui/mixins'
import { useProps, useLocale, booleanProp, booleanStringProp, sizeProp, stateProp, createSizeProp, createStateProp } from '@vexip-ui/config'
import { noop, doubleDigits, boundRange } from '@vexip-ui/utils'
import { CircleXmark, ClockR, ArrowRightArrowLeft } from '@vexip-ui/icons'
import { useColumn } from './helper'

import type { PropType } from 'vue'
import type { Placement } from '@vexip-ui/mixins'
import type { TimeType, TimeShortcut } from './symbol'

// const TIME_REG = /^((?:[01]?[0-9])|(?:2[0-3]))((?::[0-5]?[0-9]))?((?::[0-5]?[0-9]))?$/
const TIME_REG = /^((?:\d{1,2}))((?::\d{1,2}))?((?::\d{1,2}))?$/

export default defineComponent({
  name: 'TimePicker',
  components: {
    Button,
    Icon,
    Portal,
    TimeControl,
    TimeWheel,
    CircleXmark,
    ArrowRightArrowLeft
  },
  props: {
    size: sizeProp,
    state: stateProp,
    visible: booleanProp,
    placement: String as PropType<Placement>,
    transfer: booleanStringProp,
    format: String,
    separator: String,
    value: [String, Array] as PropType<string | string[]>,
    filler: String,
    noFiller: booleanProp,
    clearable: booleanProp,
    noAction: booleanProp,
    noArrow: booleanProp,
    pointer: booleanProp,
    candidate: Number as PropType<0 | 1 | 2 | 3>,
    steps: Array as PropType<number[]>,
    labels: Object as PropType<Partial<Record<TimeType, string>>>,
    shortcuts: Array as PropType<TimeShortcut[]>,
    isRange: booleanProp,
    disabled: booleanProp,
    transitionName: String,
    confirmText: String,
    cancelText: String,
    ctrlSteps: Array as PropType<number[]>,
    prefix: Object,
    prefixColor: String,
    suffix: Object,
    suffixColor: String,
    exchange: booleanProp,
    disableValidate: booleanProp
  },
  emits: [
    'change-col',
    'change',
    'focus',
    'blur',
    'plus',
    'minus',
    'enter',
    'cancel',
    'input',
    'clear',
    'shortcut',
    'toggle',
    'update:value',
    'update:visible'
  ],
  setup(_props, { slots, emit }) {
    const props = useProps('timePicker', _props, {
      size: createSizeProp(),
      state: createStateProp(),
      visible: false,
      placement: {
        default: 'bottom-start',
        validator: (value: Placement) => placementWhileList.includes(value)
      },
      transfer: false,
      format: 'HH:mm:ss',
      separator: ':',
      value: {
        default: '00:00:00',
        static: true
      },
      filler: {
        default: '-',
        validator: (value: string) => value.length === 1
      },
      noFiller: false,
      clearable: false,
      noAction: false,
      noArrow: false,
      pointer: false,
      candidate: {
        default: 3,
        validator: (value: number) => [0, 1, 2, 3].includes(value)
      },
      steps: () => [1, 1, 1],
      labels: () => ({}),
      shortcuts: () => [],
      isRange: false,
      disabled: false,
      transitionName: 'vxp-drop',
      confirmText: null,
      cancelText: null,
      ctrlSteps: () => [5, 5, 5],
      prefix: null,
      prefixColor: '',
      suffix: null,
      suffixColor: '',
      exchange: false,
      disableValidate: false
    })

    const validateField = inject(VALIDATE_FIELD, noop)
    const clearField = inject(CLEAR_FIELD, noop)

    const prefix = 'vxp-time-picker'
    const placement = toRef(props, 'placement')
    const transfer = toRef(props, 'transfer')
    const currentVisible = ref(props.visible)
    const focused = ref(false)
    const lastValue = ref('')
    const startState = createDateState()
    const endState = createDateState()
    const currentState = ref<'start' | 'end'>('start')

    const wrapper = useClickOutside()
    const { reference, popper, transferTo, updatePopper } = usePopper({
      placement,
      transfer,
      wrapper,
      isDrop: true
    })
    const { isHover } = useHover(reference)

    const startInput = ref<InstanceType<typeof TimeControl> | null>(null)
    const endInput = ref<InstanceType<typeof TimeControl> | null>(null)

    const className = computed(() => {
      return [
        prefix,
        'vxp-input-vars',
        `${prefix}-vars`,
        {
          [`${prefix}--disabled`]: props.disabled,
          [`${prefix}--${props.size}`]: props.size !== 'default',
          [`${prefix}--no-hour`]: !startState.enabled.hour,
          [`${prefix}--no-minute`]: !startState.enabled.minute,
          [`${prefix}--no-second`]: !startState.enabled.second,
          [`${prefix}--focused`]: focused.value,
          [`${prefix}--${props.state}`]: props.state !== 'default',
          [`${prefix}--is-range`]: props.isRange
        }
      ]
    })
    const hasPrefix = computed(() => {
      return !!(slots.prefix || props.prefix)
    })
    const currentValue = computed(() => {
      const values = [startState, endState].map(state => {
        return Object.values(state.timeValue).map(doubleDigits).join(':')
      })

      return props.isRange ? values : values[0]
    })

    parseValue(props.value, false)

    watch(
      () => props.value,
      value => {
        parseValue(value)
      }
    )
    watch(() => props.format, parseFormat, { immediate: true })
    watch(
      () => props.visible,
      value => {
        currentVisible.value = value
      }
    )
    watch(currentVisible, value => {
      if (value) {
        updatePopper()
      } else {
        emitChange()
      }

      emit('toggle', value)
      emit('update:visible', value)
    })
    watch(focused, value => {
      if (value) {
        emit('focus')
      } else {
        emit('blur')
      }
    })
    watch(
      () => startState.column,
      value => {
        if (currentVisible.value) {
          emit('change-col', value)
        }
      }
    )
    watch(
      () => endState.column,
      value => {
        if (currentVisible.value) {
          emit('change-col', value)
        }
      }
    )

    function createDateState() {
      const { currentColumn, enabled, resetColumn, enterColumn } = useColumn([
        'hour',
        'minute',
        'second'
      ] as TimeType[])

      const timeValue = reactive({
        hour: 0,
        minute: 0,
        second: 0
      })
      const activated = reactive({
        hour: false,
        minute: false,
        second: false
      })

      return reactive({
        column: currentColumn,
        enabled,
        activated,
        timeValue,
        resetColumn,
        enterColumn
      })
    }

    function getCurrentState() {
      return currentState.value === 'start' ? startState : endState
    }

    function parseValue(value: string | string[], updateActivated = true) {
      if (!Array.isArray(value)) {
        value = [value, value]
      }

      for (let i = 0; i < 2; ++i) {
        const match = TIME_REG.exec(value[i])
        const state = i === 0 ? startState : endState
        const { activated, timeValue } = state

        if (match) {
          if (updateActivated) {
            activated.hour = !!match[1]
            activated.minute = !!match[2]
            activated.second = !!match[3]
          }

          const hour = parseInt(match[1])
          const minute = match[2] ? parseInt(match[2].slice(1)) : 0
          const second = match[3] ? parseInt(match[3].slice(1)) : 0

          const date = new Date(2000, 1, 1, hour, minute, second)

          timeValue.hour = date.getHours()
          timeValue.minute = date.getMinutes()
          timeValue.second = date.getSeconds()
        } else {
          timeValue.hour = 0
          timeValue.minute = 0
          timeValue.second = 0

          if (updateActivated) {
            activated.hour = false
            activated.minute = false
            activated.second = false
          }
        }

        if (!props.isRange) break
      }
    }

    function parseFormat() {
      [startState, endState].forEach(state => {
        state.enabled.hour = props.format.includes('H')
        state.enabled.minute = props.format.includes('m')
        state.enabled.second = props.format.includes('s')
      })
    }

    function toggleActivated(value: boolean) {
      [startState, endState].forEach(state => {
        (Object.keys(state.activated) as TimeType[]).forEach(type => {
          state.activated[type] = value
        })
      })
    }

    function getStringValue() {
      return Array.isArray(currentValue.value) ? currentValue.value.join('|') : currentValue.value
    }

    function emitChange() {
      if (lastValue.value !== getStringValue()) {
        lastValue.value = getStringValue()

        toggleActivated(true)

        emit('change', currentValue.value)
        emit('update:value', currentValue.value)

        if (!props.disableValidate) {
          validateField()
        }
      }
    }

    function handleFocused() {
      if (props.disabled) return

      focused.value = true

      window.setTimeout(() => {
        if (focused.value) {
          if (currentState.value === 'start') {
            startInput.value?.focus()
          } else {
            endInput.value?.focus()
          }
        }
      }, 120)
    }

    function handleTirggerClick() {
      if (props.disabled) return

      currentVisible.value = true

      handleFocused()
    }

    function finishInput() {
      currentVisible.value = false
      focused.value = false

      startState.resetColumn()
      endState.resetColumn()
    }

    function handleClear() {
      if (props.clearable) {
        lastValue.value = ''
        finishInput()
        nextTick(() => {
          parseValue('')
          emit('clear')
          emit('change', currentValue.value)
          emit('update:value', currentValue.value)
          clearField()

          nextTick(() => {
            toggleActivated(false)
          })
        })
      }
    }

    function handleWheelChange(type: TimeType) {
      if (currentVisible.value) {
        getCurrentState().activated[type] = true
      }
    }

    function handleInputFocus(type: TimeType) {
      getCurrentState().column = type
    }

    function handleInput(value: number) {
      const state = getCurrentState()

      handleInputNumber(state.column, value)

      if (state.column !== 'second' && state.timeValue[state.column] >= 10) {
        state.enterColumn('next', false)
      }
    }

    function handleInputNumber(type: TimeType, number: number) {
      const state = getCurrentState()
      const prev = state.timeValue[type]

      if ((props.noFiller || state.activated[type]) && prev > 0 && prev < 10) {
        state.timeValue[type] = prev * 10 + number
      } else {
        state.timeValue[type] = number
      }

      verifyValue(type)
      emit('input', type, state.timeValue[type])
    }

    function verifyValue(type: TimeType) {
      const timeValue = getCurrentState().timeValue

      timeValue[type] = boundRange(timeValue[type], 0, type === 'hour' ? 23 : 59)
      timeValue[type] = Math.round(timeValue[type] / getStep(type)) * getStep(type)
    }

    function handlePlus(ctrlKey: boolean) {
      const state = getCurrentState()
      const type = state.column

      if (state.enabled[type]) {
        state.timeValue[type] += ctrlKey ? getCtrlStep(type) : getStep(type)

        verifyValue(type)
        emit('plus', type, state.timeValue[type])
      }
    }

    function handleMinus(ctrlKey: boolean) {
      const state = getCurrentState()
      const type = state.column

      if (state.enabled[type]) {
        state.timeValue[type] -= ctrlKey ? getCtrlStep(type) : getStep(type)

        verifyValue(type)
        emit('minus', type, state.timeValue[type])
      }
    }

    function getStep(type: TimeType) {
      return props.steps[type === 'hour' ? 0 : type === 'minute' ? 1 : 2] || 1
    }

    function getCtrlStep(type: TimeType) {
      return props.ctrlSteps[type === 'hour' ? 0 : type === 'minute' ? 1 : 2] || 1
    }

    function handleEnter() {
      finishInput()
      emit('enter')
    }

    function handleCancel() {
      parseValue(props.value)
      finishInput()
      emit('cancel')
    }

    function handleShortcut(index: number) {
      let { value, name } = props.shortcuts[index]

      if (typeof value === 'function') {
        value = value()
      }

      parseValue(value)
      emit('shortcut', name, value)
      finishInput()
    }

    function toggleCurrentState(type: 'start' | 'end') {
      currentState.value = type
    }

    function enterColumn(type: 'prev' | 'next') {
      if (props.isRange) {
        const state = getCurrentState()
        const currentColumn = state.column

        state.enterColumn(type, false)

        if (currentColumn === state.column) {
          const isStart = currentState.value === 'start'
          const otherState = isStart ? endState : startState

          otherState.resetColumn(type === 'prev' ? 'second' : 'hour', type === 'prev')
          toggleCurrentState(isStart ? 'end' : 'start')
        }
      } else {
        startState.enterColumn(type)
      }
    }

    function handleStartInput(type: TimeType) {
      toggleCurrentState('start')
      handleInputFocus(type)
    }

    function handleEndInput(type: TimeType) {
      toggleCurrentState('end')
      handleInputFocus(type)
    }

    function exchangeValue() {
      (Object.keys(startState.timeValue) as TimeType[]).forEach(type => {
        const temp = endState.timeValue[type]
        endState.timeValue[type] = startState.timeValue[type]
        startState.timeValue[type] = temp
      })
    }

    function handleExchangeClick(event: MouseEvent) {
      if (props.exchange) {
        !currentVisible.value && event.stopPropagation()
        exchangeValue()
      }
    }

    return {
      ClockR,

      props,
      prefixCls: prefix,
      locale: useLocale('timePicker'),
      isHover,
      currentVisible,
      focused,
      transferTo,
      lastValue,
      startState,
      endState,
      currentState,
      currentValue,

      className,
      hasPrefix,

      wrapper,
      reference,
      popper,
      start: startInput,
      end: endInput,

      handleFocused,
      handleTirggerClick,
      handleClear,
      handleShortcut,
      handleWheelChange,
      handleInputFocus,
      handleInput,
      handlePlus,
      handleMinus,
      handleEnter,
      finishInput,
      handleCancel,
      toggleCurrentState,
      enterColumn,
      handleStartInput,
      handleEndInput,
      handleExchangeClick,

      updatePopper
    }
  }
})
</script>
