<template>
  <div
    ref="wrapper"
    :class="className"
    :style="style"
    @mousedown="handleMouseDown"
    @wheel.exact="handleWheel($event, 'vertical')"
    @wheel.shift="handleWheel($event, 'horizontal')"
  >
    <div
      ref="content"
      :class="wrapperClass"
      :style="props.scrollStyle"
      @scroll.exact="handleScroll($event, 'vertical')"
      @scroll.shift="handleScroll($event, 'horizontal')"
    >
      <slot></slot>
    </div>
    <Scrollbar
      v-if="props.useXBar"
      ref="xBar"
      placement="bottom"
      :class="[`${prefix}__bar--horizontal`, props.barClass]"
      :fade="props.barFade"
      :bar-length="xBarLength"
      :disabled="!enableXScroll"
      :appear="props.appear"
      :duration="props.barDuration"
      :use-track="props.useBarTrack"
      @scroll-start="handleBarScrollStart('horizontal')"
      @scroll="handleXBarScroll"
      @scroll-end="handleBarScrollEnd('horizontal')"
    ></Scrollbar>
    <Scrollbar
      v-if="props.useYBar"
      ref="yBar"
      placement="right"
      :class="[`${prefix}__bar--vertical`, props.barClass]"
      :fade="props.barFade"
      :bar-length="yBarLength"
      :disabled="!enableYScroll"
      :appear="props.appear"
      :duration="props.barDuration"
      :use-track="props.useBarTrack"
      @scroll-start="handleBarScrollStart('vertical')"
      @scroll="handleYBarScroll"
      @scroll-end="handleBarScrollEnd('vertical')"
    ></Scrollbar>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, watch, toRef, nextTick } from 'vue'
import { Scrollbar } from '@/components/scrollbar'
import { useProps, booleanProp, booleanNumberProp } from '@vexip-ui/config'
import { USE_TOUCH, isTrue, createEventEmitter } from '@vexip-ui/utils'
import { useScrollWrapper } from './mixins'

import type { PropType } from 'vue'
import type { EventHandler } from '@vexip-ui/utils'
import type { ScrollMode, ClassType } from './symbol'

const scrollModes = Object.freeze<ScrollMode>(['horizontal', 'vertical', 'both'])

const MOVE_EVENT = 'mousemove'
const UP_EVENT = 'mouseup'

export default defineComponent({
  name: 'NativeScroll',
  components: {
    Scrollbar
  },
  props: {
    scrollClass: [String, Object] as PropType<ClassType>,
    scrollStyle: Object,
    mode: String as PropType<ScrollMode>,
    width: [Number, String],
    height: [Number, String],
    disabled: booleanProp,
    pointer: booleanProp,
    scrollX: Number,
    scrollY: Number,
    useXBar: booleanProp,
    useYBar: booleanProp,
    barFade: Number,
    barClass: [String, Object] as PropType<ClassType>,
    autoplay: booleanNumberProp,
    playWaiting: Number,
    onBeforeScroll: Function as PropType<(payload: { signX: number, signY: number }) => boolean>,
    appear: booleanProp,
    barDuration: Number,
    useBarTrack: booleanProp
  },
  emits: [
    'x-enable-change',
    'y-enable-change',
    'wheel',
    'scroll-start',
    'scroll',
    'scroll-end',
    'bar-scroll-start',
    'bar-scroll-end'
  ],
  setup(_props, { emit }) {
    const props = useProps('nativeScroll', _props, {
      scrollClass: null,
      scrollStyle: () => ({}),
      mode: {
        default: 'vertical' as ScrollMode,
        validator: (value: ScrollMode) => scrollModes.includes(value)
      },
      width: '',
      height: '',
      disabled: false,
      pointer: false,
      scrollX: {
        default: 0,
        static: true
      },
      scrollY: {
        default: 0,
        static: true
      },
      useXBar: false,
      useYBar: false,
      barFade: 1500,
      barClass: null,
      autoplay: false,
      playWaiting: 500,
      onBeforeScroll: {
        default: null,
        isFunc: true
      },
      appear: false,
      barDuration: null,
      useBarTrack: false
    })

    const emitter = createEventEmitter()

    const prefix = 'vxp-native-scroll'
    const usingBar = ref(false)
    const scrolling = ref(false)

    const {
      contentElement,

      currentScroll,
      percentX,
      percentY,
      xScrollLimit,
      yScrollLimit,
      enableXScroll,
      enableYScroll,
      xBarLength,
      yBarLength,

      setScrollX,
      setScrollY,
      computePercent,
      refresh,
      scrollTo,
      scrollBy,
      scrollToElement
    } = useScrollWrapper({
      mode: toRef(props, 'mode'),
      disabled: toRef(props, 'disabled'),
      appear: toRef(props, 'appear'),
      width: toRef(props, 'width'),
      height: toRef(props, 'height'),
      scrollX: toRef(props, 'scrollX'),
      scrollY: toRef(props, 'scrollY'),
      onBeforeRefresh: stopAutoplay,
      onAfterRefresh: startAutoplay
    })

    /* autoplay */
    const canPlay = ref(false)

    const canAutoplay = computed(() => {
      return (
        props.mode !== 'both' &&
        (isTrue(props.autoplay) || props.autoplay > 1000) &&
        ((props.mode === 'horizontal' && enableXScroll.value) ||
          (props.mode === 'vertical' && enableYScroll.value))
      )
    })

    watch(
      () => props.autoplay,
      () => {
        stopAutoplay()
        nextTick(startAutoplay)
      }
    )
    watch(
      () => props.playWaiting,
      () => {
        stopAutoplay()
        nextTick(startAutoplay)
      }
    )

    let playTimer: number
    let startTimer: number
    let endTimer: number

    function startAutoplay() {
      if (!canAutoplay.value || !contentElement.value) return

      stopAutoplay()

      const mode = props.mode
      const distance = mode === 'horizontal' ? 'offsetWidth' : 'offsetHeight'
      const limit = mode === 'horizontal' ? xScrollLimit : yScrollLimit
      const prop = mode === 'horizontal' ? 'x' : 'y'
      const waiting = props.playWaiting < 20 ? 20 : props.playWaiting
      const setScroll = mode === 'horizontal' ? setScrollX : setScrollY

      let playSpeed = 0.5

      if (typeof props.autoplay === 'number') {
        playSpeed = (contentElement.value[distance] / props.autoplay) * 16
      }

      const scroll = () => {
        setScroll(currentScroll[prop] + playSpeed)

        if (currentScroll[prop] >= limit.value) {
          setScroll(limit.value)
          canPlay.value = false

          computePercent()
          syncBarScroll()

          endTimer = window.setTimeout(() => {
            scrollTo(0, 0, 500)

            startTimer = window.setTimeout(() => {
              canPlay.value = true
              scroll()
            }, 500 + waiting)
          }, waiting)
        } else {
          computePercent()
          syncBarScroll()

          if (canPlay.value) {
            requestAnimationFrame(scroll)
          }
        }
      }

      playTimer = window.setTimeout(() => {
        canPlay.value = true
        scroll()
      }, waiting)
    }

    function stopAutoplay() {
      canPlay.value = false

      window.clearTimeout(playTimer)
      window.window.clearTimeout(startTimer)
      window.clearTimeout(endTimer)
    }
    /* autoplay */

    const className = computed(() => {
      return [prefix, `${prefix}--${props.mode}`]
    })
    const style = computed(() => {
      const { width, height } = props

      return {
        width: width
          ? typeof width === 'string'
            ? Number.isNaN(Number(width))
              ? width
              : `${Number(width)}px`
            : `${width}px`
          : undefined,
        height: height
          ? typeof height === 'string'
            ? Number.isNaN(Number(height))
              ? height
              : `${Number(height)}px`
            : `${height}px`
          : undefined
      }
    })
    const wrapperClass = computed(() => {
      return [
        `${prefix}__wrapper`,
        props.scrollClass,
        {
          [`${prefix}__wrapper--scrolling`]: scrolling.value,
          [`${prefix}__wrapper--using-bar`]: usingBar.value
        }
      ]
    })

    watch(enableXScroll, value => {
      emit('x-enable-change', value)
    })
    watch(enableYScroll, value => {
      emit('y-enable-change', value)
    })

    const xBar = ref<InstanceType<typeof Scrollbar> | null>(null)
    const yBar = ref<InstanceType<typeof Scrollbar> | null>(null)

    function syncBarScroll() {
      xBar.value?.handleScroll(percentX.value)
      yBar.value?.handleScroll(percentY.value)
    }

    function handleMouseDown(event: MouseEvent) {
      if (!props.pointer || event.button !== 0 || USE_TOUCH) {
        return false
      }

      handlePointerDown(event)
    }

    // 记录滚动开始位置
    let xScrollStartAt = 0
    let yScrollStartAt = 0

    // 记录滚动开始鼠标位置
    let cursorXPosition = 0
    let cursorYPosition = 0

    function handlePointerDown(event: MouseEvent) {
      if (!enableXScroll.value && !enableYScroll.value) {
        return false
      }

      event.preventDefault()
      prepareScroll()

      xScrollStartAt = currentScroll.x
      yScrollStartAt = currentScroll.y
      cursorXPosition = event.clientX
      cursorYPosition = event.clientY

      document.addEventListener(MOVE_EVENT, handlePointerMove)
      document.addEventListener(UP_EVENT, handlePointerUp)

      emit('scroll-start', {
        clientX: currentScroll.x,
        clientY: currentScroll.y,
        percentX: percentX.value,
        percentY: percentY.value
      })
    }

    function handlePointerMove(event: MouseEvent) {
      event.stopPropagation()
      event.preventDefault()

      const signX = event.clientX - cursorXPosition > 0 ? 1 : -1
      const signY = event.clientY - cursorYPosition > 0 ? 1 : -1

      if (props.onBeforeScroll?.({ signX, signY }) === false) {
        return false
      }

      scrolling.value = true

      if (enableXScroll.value) {
        setScrollX(xScrollStartAt - (event.clientX - cursorXPosition))
      }

      if (enableYScroll.value) {
        setScrollY(yScrollStartAt - (event.clientY - cursorYPosition))
      }

      computePercent()
      syncBarScroll()
      emitScrollEvent()
    }

    function handlePointerUp() {
      document.removeEventListener(MOVE_EVENT, handlePointerMove)
      document.removeEventListener(UP_EVENT, handlePointerUp)
      emitScrollEvent()
      emit('scroll-end', {
        clientX: currentScroll.x,
        clientY: currentScroll.y,
        percentX: percentX.value,
        percentY: percentY.value
      })
      startAutoplay()
    }

    function handleWheel(event: WheelEvent, type: 'vertical' | 'horizontal') {
      const isVerticalScroll = enableYScroll.value && type === 'vertical'
      const isHorizontalScroll = enableXScroll.value && type === 'horizontal'
      const sign = event.deltaY > 0 ? 1 : -1

      if (
        (isVerticalScroll || isHorizontalScroll) &&
        props.onBeforeScroll?.({ signX: sign, signY: sign }) !== false
      ) {
        const maxLimit = isVerticalScroll ? yScrollLimit.value : xScrollLimit.value
        const scroll = isVerticalScroll ? currentScroll.y : currentScroll.x

        if (sign > 0 ? scroll < maxLimit : scroll > 0) {
          event.stopPropagation()

          return false
        }
      }
    }

    function handleScroll(event: Event, type: 'vertical' | 'horizontal') {
      event.stopPropagation()
      event.preventDefault()

      if (contentElement.value) {
        const signX = contentElement.value.scrollLeft - currentScroll.x > 0 ? 1 : -1
        const signY = contentElement.value.scrollTop - currentScroll.y > 0 ? 1 : -1

        if (props.onBeforeScroll?.({ signX, signY }) === false) {
          contentElement.value.scrollTop = currentScroll.y
          contentElement.value.scrollLeft = currentScroll.x

          return
        }

        currentScroll.y = contentElement.value.scrollTop
        currentScroll.x = contentElement.value.scrollLeft
      }

      computePercent()
      syncBarScroll()
      emitScrollEvent(type)
    }

    function prepareScroll() {
      stopAutoplay()
    }

    function handleBarScrollStart(type: 'vertical' | 'horizontal') {
      usingBar.value = true
      emit('bar-scroll-start', type)
    }

    function handleBarScrollEnd(type: 'vertical' | 'horizontal') {
      usingBar.value = false
      emit('bar-scroll-end', type)
    }

    function handleXBarScroll(percent: number) {
      percentX.value = percent
      setScrollX((percent * xScrollLimit.value) / 100)
      emitScrollEvent('horizontal')
    }

    function handleYBarScroll(percent: number) {
      percentY.value = percent
      setScrollY((percent * yScrollLimit.value) / 100)
      emitScrollEvent('vertical')
    }

    function emitScrollEvent(type?: 'vertical' | 'horizontal') {
      emit('scroll', {
        type,
        clientX: currentScroll.x,
        clientY: currentScroll.y,
        percentX: percentX.value,
        percentY: percentY.value
      })
      emitter.emit('scroll', {
        type,
        clientX: currentScroll.x,
        clientY: currentScroll.y,
        percentX: percentX.value,
        percentY: percentY.value
      })
    }

    function getXScrollLimit() {
      return [0, xScrollLimit.value]
    }

    function getYScrollLimit() {
      return [0, yScrollLimit.value]
    }

    function addScrollListener(listener: EventHandler) {
      emitter.on('scroll', listener)
    }

    function removeScrollListener(listener: EventHandler) {
      emitter.off('scroll', listener)
    }

    return {
      props,
      prefix,
      percentX,
      percentY,
      currentScroll,
      xScrollLimit,
      yScrollLimit,

      className,
      style,
      wrapperClass,
      xBarLength,
      yBarLength,
      enableXScroll,
      enableYScroll,

      wrapper: ref<HTMLElement | null>(null),
      content: contentElement,
      xBar,
      yBar,

      handleMouseDown,
      handleScroll,
      handleWheel,
      handleBarScrollStart,
      handleBarScrollEnd,
      handleXBarScroll,
      handleYBarScroll,

      refresh,
      scrollTo,
      scrollBy,
      scrollToElement,
      getXScrollLimit,
      getYScrollLimit,
      addScrollListener,
      removeScrollListener
    }
  }
})
</script>
