<template>
  <transition-group
    v-if="props.type === 'thumbnail'"
    tag="ul"
    :appear="props.selectToAdd"
    :name="`${prefix}-list-transition`"
    :class="[`${prefix}__files`, `${prefix}-vars`]"
    :style="props.style"
  >
    <UploadFile
      v-for="item in props.files"
      :key="item.id"
      :file="item"
      :icon-renderer="props.iconRenderer"
      :list-type="props.type"
      :loading-text="props.loadingText"
      :select-to-add="props.selectToAdd"
      @delete="$emit('delete', $event)"
      @preview="$emit('preview', $event)"
    >
      <template #default="{ file, status: _status, percentage }">
        <slot
          name="item"
          :file="file"
          :status="_status"
          :percentage="percentage"
        ></slot>
      </template>
      <template #icon="{ file }">
        <slot name="icon" :file="file"></slot>
      </template>
    </UploadFile>
  </transition-group>
  <ul v-else :class="`${prefix}__files`" :style="props.style">
    <UploadFile
      v-for="item in props.files"
      :key="item.id"
      :file="item"
      :icon-renderer="props.iconRenderer"
      :list-type="props.type"
      :loading-text="props.loadingText"
      :select-to-add="props.selectToAdd"
      @delete="$emit('delete', $event)"
      @preview="$emit('preview', $event)"
    >
      <template #default="{ file, status: _status, percentage }">
        <slot
          name="item"
          :file="file"
          :status="_status"
          :percentage="percentage"
        ></slot>
      </template>
      <template #icon="{ file }">
        <slot name="icon" :file="file"></slot>
      </template>
    </UploadFile>
  </ul>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import UploadFile from './upload-file.vue'
import { useProps, booleanProp } from '@vexip-ui/config'
import { uploadListTypes } from './symbol'

import type { PropType, StyleValue } from 'vue'
import type { UploadListType, RenderFn, FileState } from './symbol'

export default defineComponent({
  name: 'UploadList',
  components: {
    UploadFile
  },
  props: {
    files: Array as PropType<FileState[]>,
    selectToAdd: booleanProp,
    iconRenderer: Function as PropType<RenderFn>,
    type: String as PropType<UploadListType>,
    loadingText: String,
    style: [String, Object] as PropType<StyleValue>
  },
  emits: ['preview', 'delete'],
  setup(_props) {
    const props = useProps('uploadList', _props, {
      files: {
        default: () => [],
        static: true
      },
      selectToAdd: false,
      iconRenderer: {
        default: null,
        isFunc: true
      },
      type: {
        default: 'name' as UploadListType,
        validator: (value: UploadListType) => uploadListTypes.includes(value)
      },
      loadingText: null,
      style: null
    })

    return {
      props,
      prefix: 'vxp-upload'
    }
  }
})
</script>
