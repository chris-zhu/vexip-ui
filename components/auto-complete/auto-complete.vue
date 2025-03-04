<template>
  <Select
    ref="select"
    v-model:visible="visible"
    :class="prefixCls"
    :list-class="`${prefixCls}__list`"
    :value="currentValue"
    :size="props.size"
    :state="props.state"
    :clearable="props.clearable"
    :transition-name="props.transitionName"
    :disabled="props.disabled"
    :transfer="props.transfer"
    :placement="props.placement"
    :prefix-color="props.prefixColor"
    :suffix-color="props.suffixColor"
    :no-suffix="!hasSuffix"
    :placeholder="props.placeholder"
    :options="props.options"
    :value-key="props.valueKey"
    :label-key="props.labelKey"
    @toggle="handleToggle"
    @select="handleSelect"
    @clear="handleClear"
  >
    <template v-if="hasPrefix" #prefix>
      <slot name="prefix">
        <Icon :icon="props.prefix"></Icon>
      </slot>
    </template>
    <template #control>
      <slot
        name="control"
        :value="currentValue"
        :on-input="handleInput"
        :on-change="handleChange"
        :on-enter="handleEnter"
        :on-clear="handleClear"
      >
        <input
          ref="control"
          :class="`${prefixCls}__input`"
          :value="inputValue"
          :autofocus="props.autofocus"
          :spellcheck="props.spellcheck"
          :disabled="props.disabled"
          :placeholder="props.placeholder ?? locale.placeholder"
          autocomplete="off"
          @input="handleInput"
          @change="handleInputChange"
          @keyup.enter="handleEnter"
          @keydown="handleKeyDown"
        />
      </slot>
    </template>
    <template v-if="hasSuffix" #suffix>
      <slot name="suffix">
        <Icon :icon="props.suffix"></Icon>
      </slot>
    </template>
    <template #default="{ option, index, selected }">
      <slot
        :option="option"
        :index="index"
        :selected="selected"
        :handle-select="handleSelect"
      ></slot>
    </template>
  </Select>
</template>

<script lang="ts">
import { defineComponent, ref, computed, inject, watch, watchEffect } from 'vue'
import { Icon } from '@/components/icon'
import { Select } from '@/components/select'
import { VALIDATE_FIELD, CLEAR_FIELD } from '@/components/form-item'
import { placementWhileList } from '@vexip-ui/mixins'
import {
  useProps,
  useLocale,
  booleanProp,
  booleanStringProp,
  sizeProp,
  stateProp,
  createSizeProp,
  createStateProp
} from '@vexip-ui/config'
import { isNull, noop } from '@vexip-ui/utils'

import type { PropType } from 'vue'
import type { Placement } from '@vexip-ui/mixins'
import type { OptionState } from '@/components/option'
import type { RawOption } from '@/components/select'

export default defineComponent({
  name: 'AutoComplete',
  components: {
    Icon,
    Select
  },
  props: {
    size: sizeProp,
    state: stateProp,
    transfer: booleanStringProp,
    value: [String, Number],
    options: Array as PropType<RawOption[]>,
    filter: {
      type: [Boolean, Function] as PropType<
        | boolean
        | ((value: string | number, options: { label: string, value: string | number }) => boolean)
      >,
      default: null
    },
    prefix: Object,
    prefixColor: String,
    suffix: Object,
    suffixColor: String,
    placeholder: String,
    disabled: booleanProp,
    transitionName: String,
    dropDisabled: booleanProp,
    placement: String as PropType<Placement>,
    disableValidate: booleanProp,
    clearable: booleanProp,
    ignoreCase: booleanProp,
    autofocus: booleanProp,
    spellcheck: booleanProp,
    valueKey: String,
    labelKey: String
  },
  emits: [
    'select',
    'input',
    'change',
    'toggle',
    'enter',
    'clear',
    'update:value'
  ],
  setup(_props, { slots, emit }) {
    const props = useProps('autoComplete', _props, {
      size: createSizeProp(),
      state: createStateProp(),
      transfer: false,
      value: {
        default: '',
        static: true
      },
      options: {
        default: () => [],
        static: true
      },
      filter: false,
      prefix: null,
      prefixColor: '',
      suffix: null,
      suffixColor: '',
      placeholder: null,
      disabled: false,
      transitionName: 'vxp-drop',
      dropDisabled: false,
      placement: {
        default: 'bottom',
        validator: (value: Placement) => placementWhileList.includes(value)
      },
      disableValidate: false,
      clearable: false,
      ignoreCase: false,
      autofocus: false,
      spellcheck: false,
      valueKey: 'value',
      labelKey: 'label'
    })

    const validateField = inject(VALIDATE_FIELD, noop)
    const clearField = inject(CLEAR_FIELD, noop)

    const prefix = 'vxp-auto-complete'
    const currentValue = ref(props.value)
    const currentIndex = ref(-1)
    const visible = ref(false)

    const select = ref<InstanceType<typeof Select> | null>(null)
    const control = ref<HTMLInputElement | null>(null)

    let changed = false
    // eslint-disable-next-line vue/no-setup-props-destructure
    let lastValue = props.value

    const optionsStates = computed<OptionState[]>(() => {
      return select.value?.optionStates ?? []
    })
    const filteredOptions = computed(() => {
      return optionsStates.value.filter(({ hidden }) => !hidden)
    })
    const inputValue = computed(() => {
      const hittingOption = filteredOptions.value[currentIndex.value]

      if (hittingOption) {
        return hittingOption.label || hittingOption.value?.toString()
      } else if (select.value) {
        if (select.value.currentVisible) {
          return currentValue.value?.toString()
        }

        const currentOption = filteredOptions.value.find(
          ({ value }) => value === currentValue.value
        )

        return currentOption ? currentOption.label : currentValue.value?.toString()
      }

      return currentValue.value?.toString()
    })
    const hasPrefix = computed(() => {
      return !!(slots.prefix || props.prefix)
    })
    const hasSuffix = computed(() => {
      return !!(slots.suffix || props.suffix)
    })

    watch(
      () => props.value,
      value => {
        currentValue.value = value
        lastValue = value
      }
    )
    watch(currentIndex, computeHittding)
    watch(visible, value => {
      if (!value) {
        currentIndex.value = -1
        control.value?.blur()
      } else {
        control.value?.focus()
      }
    })

    watchEffect(() => {
      if (props.filter) {
        const value = currentValue.value

        if (isNull(value)) {
          optionsStates.value.forEach(state => {
            state.hidden = false
          })
        } else {
          if (typeof props.filter === 'function') {
            const filter = props.filter

            optionsStates.value.forEach(state => {
              state.hidden = !filter(value, { label: state.label, value: state.value })
            })
          } else {
            if (props.ignoreCase) {
              optionsStates.value.forEach(state => {
                state.hidden = !state.value
                  ?.toString()
                  .toLocaleLowerCase()
                  .includes(value?.toString().toLocaleLowerCase())
              })
            } else {
              optionsStates.value.forEach(state => {
                state.hidden = !state.value?.toString().includes(value?.toString())
              })
            }
          }
        }

        computeHittding()
      }
    })

    function computeHittding() {
      let index = -1

      optionsStates.value.forEach(state => {
        if (!state.hidden) {
          index += 1
          state.hitting = currentIndex.value === index
        } else {
          state.hitting = false
        }
      })
    }

    function handleSelect(value: string | number) {
      if (isNull(value)) {
        return
      }

      const prevValue = currentValue.value

      currentValue.value = value
      emit('select', value)

      if (value !== prevValue) {
        changed = true
        handleChange()
      } else {
        visible.value = false
      }
    }

    function handleInput(event: string | Event) {
      const value = typeof event === 'string' ? event : (event.target as HTMLInputElement).value

      visible.value = !props.dropDisabled
      currentValue.value = value
      changed = true

      if (currentIndex.value !== -1) {
        currentIndex.value = 0
      }

      emit('input', value)
    }

    function handleInputChange(event: string | Event) {
      const value = typeof event === 'string' ? event : (event.target as HTMLInputElement).value
      let matchedOption: OptionState | undefined

      if (props.ignoreCase) {
        const noCaseValue = value.toLocaleLowerCase()

        matchedOption = filteredOptions.value.find(option => !option.disabled && (option.value === value || option.label?.toLocaleLowerCase() === noCaseValue))
      } else {
        matchedOption = filteredOptions.value.find(option => !option.disabled && (option.value === value || option.label === value))
      }

      if (matchedOption) {
        currentValue.value = matchedOption.value
        handleChange()
      }
    }

    function handleChange() {
      if (!changed || currentValue.value === lastValue) return

      changed = false
      lastValue = currentValue.value

      emit('change', currentValue.value)
      emit('update:value', currentValue.value)

      if (!props.disableValidate) {
        validateField()
      }

      visible.value = false
      control.value?.blur()
    }

    function handleToggle() {
      testOptionCanDrop()

      emit('toggle', visible.value)

      if (!visible.value) {
        currentIndex.value = -1
      }
    }

    function testOptionCanDrop() {
      if ((!slots.default && !filteredOptions.value.length) || props.dropDisabled) {
        visible.value = false
      }
    }

    function handleKeyDown(event: KeyboardEvent) {
      const key = event.key

      if (key === 'ArrowDown' || key === 'ArrowUp') {
        event.preventDefault()

        currentIndex.value += key === 'ArrowDown' ? 1 : key === 'ArrowUp' ? -1 : 0
        currentIndex.value = Math.min(
          Math.max(-1, currentIndex.value),
          filteredOptions.value.length - 1
        )
      } else if (!['Enter', 'ArrowLeft', 'ArrowRight'].includes(key)) {
        // 进行了其他按键则重置
        currentIndex.value = -1
      }
    }

    function handleEnter() {
      if (filteredOptions.value.length) {
        handleSelect(
          filteredOptions.value[currentIndex.value === -1 ? 0 : currentIndex.value].value
        )
      } else {
        handleChange()
      }

      emit('enter', currentValue.value)
      control.value?.blur()
      visible.value = false
    }

    function handleClear() {
      if (props.clearable) {
        const prevValue = currentValue.value

        currentValue.value = ''
        visible.value = false

        if (!isNull(prevValue) && prevValue !== currentValue.value) {
          changed = true
        }

        handleChange()
        emit('clear')
        clearField()
      }
    }

    return {
      props,
      prefixCls: prefix,
      locale: useLocale('input'),
      currentValue,
      currentIndex,
      visible,

      inputValue,
      hasPrefix,
      hasSuffix,

      select,
      control,

      handleSelect,
      handleInput,
      handleInputChange,
      handleChange,
      handleToggle,
      handleKeyDown,
      handleEnter,
      handleClear
    }
  }
})
</script>
