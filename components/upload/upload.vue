<template>
  <div :class="className">
    <div
      :class="{
        [`${prefix}__control`]: true,
        [`${prefix}__control--drag-over`]: isDragOver
      }"
      @click="handleClick"
      @drop.prevent="handleDrop"
      @dragover.prevent="handleDragEnter"
      @dragleave.prevent="handleDragLeave"
    >
      <input
        v-if="!props.disabledClick"
        ref="input"
        type="file"
        :class="`${prefix}__input`"
        :multiple="props.multiple"
        :accept="acceptString"
        :webkitdirectory="props.directory"
        @change="handleInputChange"
      />
      <slot :is-drag-over="(props.allowDrag || props.disabledClick) && isDragOver">
        <template v-if="!props.allowDrag && !props.disabledClick">
          <Button :icon="Upload">
            {{ props.buttonLable ?? locale.upload }}
          </Button>
          <slot name="tip">
            <p v-if="props.tip" :class="`${prefix}__tip`">
              {{ props.tip }}
            </p>
          </slot>
        </template>
        <div v-else :class="`${prefix}__drag-pane`">
          <Icon :class="`${prefix}__cloud`" :scale="4">
            <CloudArrowUp></CloudArrowUp>
          </Icon>
          <slot name="tip">
            <p :class="`${prefix}__tip`">
              {{ props.tip || locale.dragOrClick }}
            </p>
          </slot>
        </div>
      </slot>
    </div>
    <UploadList
      v-if="!props.hiddenFiles"
      :files="renderFiles"
      :select-to-add="props.selectToAdd"
      :type="props.listType"
      :icon-renderer="props.iconRenderer"
      :loading-text="props.loadingText"
      :style="{
        [props.selectToAdd ? 'marginBottom' : 'marginTop']:
          !props.hiddenFiles && renderFiles.length ? '0.5em' : null
      }"
      @delete="deleteFile"
      @preview="$emit('preview', $event)"
    ></UploadList>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue'
import { Button } from '@/components/button'
import { Icon } from '@/components/icon'
import UploadList from './upload-list.vue'
import { upload } from './request'
import { useProps, useLocale, booleanProp } from '@vexip-ui/config'
import { isFalse, isFunction, isPromise, randomString } from '@vexip-ui/utils'
import { CloudArrowUp, Upload } from '@vexip-ui/icons'
import { UploadStatusType, uploadListTypes } from './symbol'

import type { PropType, Ref } from 'vue'
import type {
  UploadListType,
  BeforeFn,
  RenderFn,
  HttpError,
  SourceFile,
  FileState,
  DirectoryEntity
} from './symbol'

export default defineComponent({
  name: 'Upload',
  components: {
    Button,
    Icon,
    UploadList,
    CloudArrowUp
  },
  props: {
    url: String,
    multiple: booleanProp,
    tip: String,
    accept: [String, Array] as PropType<string | string[]>,
    filter: [String, Array] as PropType<string | string[]>,
    maxSize: Number,
    field: String,
    data: Object as PropType<Record<string, string | Blob>>,
    headers: Object as PropType<Record<string, string>>,
    withCredentials: booleanProp,
    manual: booleanProp,
    hiddenFiles: booleanProp,
    countLimit: Number,
    allowDrag: booleanProp,
    onBeforeUpload: Function as PropType<BeforeFn>,
    onBeforeSelect: Function as PropType<BeforeFn>,
    iconRenderer: Function as PropType<RenderFn>,
    selectToAdd: booleanProp,
    listType: String as PropType<UploadListType>,
    block: booleanProp,
    loadingText: String,
    directory: booleanProp,
    pathField: String,
    disabledClick: booleanProp,
    buttonLable: String
  },
  emits: [
    'exceed',
    'change',
    'filter-error',
    'size-error',
    'delete',
    'progress',
    'success',
    'error',
    'preview'
  ],
  setup(_props, { emit }) {
    const props = useProps('upload', _props, {
      url: {
        default: '',
        static: true
      },
      multiple: false,
      tip: '',
      accept: null,
      filter: '',
      maxSize: {
        default: null,
        validator: (value: number) => value >= 0
      },
      field: 'file',
      data: () => ({}),
      headers: () => ({}),
      withCredentials: false,
      manual: false,
      hiddenFiles: false,
      countLimit: {
        default: 0,
        validator: (value: number) => value >= 0
      },
      allowDrag: false,
      onBeforeUpload: {
        default: null,
        isFunc: true
      },
      onBeforeSelect: {
        default: null,
        isFunc: true
      },
      iconRenderer: {
        default: null,
        isFunc: true
      },
      selectToAdd: false,
      listType: {
        default: 'name' as UploadListType,
        validator: (value: UploadListType) => uploadListTypes.includes(value)
      },
      block: false,
      loadingText: null,
      directory: false,
      pathField: 'path',
      disabledClick: false,
      buttonLable: null
    })

    const prefix = 'vxp-upload'
    const fileStates = ref([]) as Ref<FileState[]>
    const isDragOver = ref(false)

    const input = ref<HTMLInputElement | null>(null)

    const className = computed(() => {
      return [
        prefix,
        `${prefix}-vars`,
        `${prefix}--type-${props.listType}`,
        {
          [`${prefix}--multiple`]: props.multiple,
          [`${prefix}--drag`]: props.allowDrag,
          [`${prefix}--to-add`]: props.selectToAdd,
          [`${prefix}--block`]: props.block,
          [`${prefix}--drag-only`]: props.disabledClick
        }
      ]
    })
    const acceptString = computed(() => {
      const accept = props.accept

      return accept && (typeof accept === 'string' ? accept : accept.join())
    })
    const renderFiles = computed(() => {
      return fileStates.value.filter(item => item.status !== UploadStatusType.DELETE)
    })

    function handleClick() {
      !props.disabledClick && input.value?.click()
    }

    function handleInputChange(event: Event) {
      const target = event.target as HTMLInputElement

      if (target?.files) {
        handleFilesChange(target.files)
      }
    }

    async function handleFilesChange(inputFiles: FileList | SourceFile[]) {
      const originFiles = Array.from(inputFiles || []) as SourceFile[]
      const files = props.selectToAdd ? fileStates.value : []

      for (const file of originFiles) {
        if (!file.path) {
          file.path = file.webkitRelativePath
        }

        if (isFunction(props.onBeforeSelect)) {
          let result = props.onBeforeSelect(file, props.selectToAdd ? getSourceFiles() : [])

          if (isPromise(result)) {
            result = await result
          }

          if (isFalse(result)) continue
        }

        let fileState = getFileStateBySource(file)

        if (fileState) {
          if (
            fileState.status !== UploadStatusType.SUCCESS &&
            fileState.status !== UploadStatusType.UPLOADING
          ) {
            fileState.status = UploadStatusType.PENDING
          }
        } else {
          fileState = createFileState(file)
        }

        if (!files.includes(fileState)) {
          files.push(fileState)
        }
      }

      const countLimit = props.countLimit

      if (countLimit > 0 && files.length > countLimit) {
        fileStates.value = files.slice(0, countLimit)

        const exceedFiles = files.slice(countLimit)

        emit(
          'exceed',
          exceedFiles.map(file => file.source),
          getSourceFiles()
        )
      } else {
        fileStates.value = files
      }

      const sourceFiles = getSourceFiles()

      syncInputFiles()
      emit('change', sourceFiles)

      if (!props.manual) {
        execute()
      }
    }

    function getFileStateBySource(file: SourceFile) {
      const { name, size, type } = file
      const path = file.path || file.webkitRelativePath

      return fileStates.value.find(({ source }) => {
        return (
          (source.path || source.webkitRelativePath) === path &&
          source.name === name &&
          source.size === size &&
          source.type === type
        )
      })
    }

    function createFileState(file: SourceFile): FileState {
      const { name, size, type } = file

      return {
        id: randomString(),
        name,
        size,
        type,
        base64: null,
        status: UploadStatusType.PENDING,
        percentage: 0,
        source: file,
        path: file.path || file.webkitRelativePath,
        xhr: null,
        response: null,
        error: null
      }
    }

    function getSourceFiles(): SourceFile[] {
      return fileStates.value.map(file => file.source)
    }

    function getFileExtension(file: FileState) {
      return file.name.split('.').pop()!.toLocaleLowerCase()
    }

    function execute() {
      if (!props.url || !verifyFiles()) {
        return false
      }

      const uploadFiles = fileStates.value.filter(
        item => item.status !== UploadStatusType.SUCCESS && item.status !== UploadStatusType.DELETE
      )
      const requests: Promise<any>[] = []

      for (const file of uploadFiles) {
        requests.push(uploadFile(file))
      }

      return Promise.all(requests).then(responses => responses.filter(response => response))
    }

    async function uploadFile(file: FileState) {
      if (isFunction(props.onBeforeUpload)) {
        let result = props.onBeforeUpload(
          file.source,
          fileStates.value
            .filter(
              item =>
                item.status !== UploadStatusType.SUCCESS && item.status !== UploadStatusType.DELETE
            )
            .map(item => item.source)
        )

        if (isPromise(result)) {
          result = await result
        }

        if (isFalse(result)) return
      }

      file.status = UploadStatusType.UPLOADING

      const { url, headers, withCredentials, data, field, pathField } = props

      return await new Promise((resolve, reject) => {
        file.xhr = upload({
          url,
          headers,
          withCredentials,
          data,
          field,
          pathField,
          file: file.source,
          onProgress: percent => {
            handleProgress(percent, file)
          },
          onSuccess: response => {
            handleSuccess(response, file)
            resolve(response)
          },
          onError: error => {
            handleError(error, file)
            reject(error)
          },
          onAbort: () => {
            resolve(null)
          }
        })
      })
    }

    function verifyFiles() {
      const limitSize = props.maxSize ? props.maxSize * 1024 : Infinity
      const filter =
        typeof props.filter === 'string'
          ? props.filter
            ? [props.filter]
            : []
          : props.filter.filter(item => item)

      for (let i = 0, len = fileStates.value.length; i < len; ++i) {
        const file = fileStates.value[i]
        const extension = getFileExtension(file)

        if (filter.length && !filter.includes(extension)) {
          emit('filter-error', file.source)

          return false
        }

        if (file.size > limitSize) {
          emit('size-error', file.source)

          return false
        }
      }

      return true
    }

    function deleteFile(file: FileState) {
      file.status = UploadStatusType.DELETE

      if (file.xhr) {
        file.xhr.abort()
      }

      syncInputFiles()
      emit('delete', file.source)
    }

    function syncInputFiles() {
      const files = fileStates.value.filter(item => item.status !== UploadStatusType.DELETE)
      const dataTransfer = new DataTransfer()

      files.forEach(item => {
        dataTransfer.items.add(item.source)
      })

      if (input.value) {
        input.value.files = dataTransfer.files
      }
    }

    // 根据源文件删除
    // function handleDelete(originFile: File) {
    //   const file = fileStates.value.find(file => file.source === originFile)

    //   if (file && file.status !== UploadStatusType.DELETE) {
    //     deleteFile(file)
    //   }
    // }

    // function clear() {
    //   fileStates.value = []

    //   if (input.value) {
    //     input.value.value = ''
    //   }

    //   emit('change', [])
    // }

    // 根据源文件修改进度
    // function progress(originFile: File, percent: number) {
    //   const file = fileStates.value.find(file => file.source === originFile)

    //   if (file && file.status === UploadStatusType.UPLOADING) {
    //     file.percentage = percent
    //   }
    // }

    // function setStatus(originFile: File, status: UploadStatusType) {
    //   if (!Object.keys(UploadStatusType).includes(status)) return

    //   const file = fileStates.value.find(file => file.source === originFile)

    //   if (file) {
    //     file.status = status
    //   }
    // }

    function handleProgress(percent: number, file: FileState) {
      if (file.status === UploadStatusType.DELETE) return

      file.percentage = percent

      emit('progress', percent, file.source)
    }

    function handleSuccess(response: any, file: FileState) {
      if (file.status === UploadStatusType.DELETE) return

      file.status = UploadStatusType.SUCCESS
      file.response = response
      file.error = null

      emit('success', response, file.source)
    }

    function handleError(error: HttpError, file: FileState) {
      if (file.status === UploadStatusType.DELETE) return

      file.status = UploadStatusType.FAIL
      file.error = error

      emit('error', error, file.source)
    }

    let dragTimer: number

    async function handleDrop(event: DragEvent) {
      if (!props.allowDrag) return

      window.clearTimeout(dragTimer)

      isDragOver.value = false

      if (event.dataTransfer) {
        const files = await collectDropFiles(event.dataTransfer)

        files.length && handleFilesChange(files)
      }
    }

    function handleDragEnter() {
      if (!props.allowDrag) return

      window.clearTimeout(dragTimer)

      isDragOver.value = true
    }

    function handleDragLeave() {
      if (!props.allowDrag) return

      dragTimer = window.setTimeout(() => {
        isDragOver.value = false
      }, 100)
    }

    async function collectDropFiles(dataTransfer: DataTransfer) {
      const { items, files } = dataTransfer

      if (!items.length) return []

      const collectedFiles: File[] = []
      const dirLoop: Array<{ dir: DirectoryEntity, prefix: string }> = []
      const processes: Promise<void>[] = []

      for (let i = 0, len = items.length; i < len; ++i) {
        const entity = items[i].webkitGetAsEntry?.()

        // 内核不支持
        if (!entity) return files

        if (entity.isFile) {
          collectedFiles.push(files[i])
        } else {
          dirLoop.push({ dir: entity as unknown as DirectoryEntity, prefix: '' })
          // directories.push(entity as unknown as DirectoryEntity)
        }
      }

      if (!props.directory || !dirLoop.length) return collectedFiles

      const fileEntries: Array<{ entry: DirectoryEntity, prefix: string }> = []

      let countLimit = props.countLimit - (props.selectToAdd ? fileStates.value.length : 0)
      countLimit = Math.round(countLimit) > 0 ? countLimit : 100

      const doProcess = () => {
        while (dirLoop.length) {
          const loop = dirLoop.shift()!
          const dir = loop.dir
          const prefix = loop.prefix ? `${loop.prefix}/${dir.name}` : dir.name
          const reader = dir.createReader()

          processes.push(
            new Promise<void>(resolve => {
              reader.readEntries(entries => {
                entries.forEach(entry => {
                  if (entry.isFile) {
                    fileEntries.push({ entry, prefix })
                  } else {
                    dirLoop.push({ dir: entry, prefix })
                  }
                })

                resolve()
              })
            })
          )
        }
      }

      while (true) {
        doProcess()
        await Promise.all(processes)

        if (!dirLoop.length || fileEntries.length >= countLimit) {
          break
        }
      }

      if (fileEntries.length > 0) {
        return collectedFiles.concat(
          await Promise.all(
            fileEntries.map(
              ({ entry, prefix }) =>
                new Promise<File>(resolve =>
                  entry.file(file => {
                    file.path = `${prefix}/${file.name}`
                    resolve(file)
                  })
                )
            )
          )
        )
      }

      return collectedFiles
    }

    return {
      Upload,

      props,
      prefix,
      locale: useLocale('upload'),
      fileStates,
      isDragOver,

      className,
      acceptString,
      renderFiles,

      input,

      handleClick,
      handleInputChange,
      deleteFile,
      handleDrop,
      handleDragEnter,
      handleDragLeave,

      execute
    }
  }
})
</script>
