<template>
  <div>
    <Button type="primary" @click="active = !active">
      打开
    </Button>
    <Modal
      v-model:active="active"
      transfer
      :width="500"
      :title="title"
      :loading="loading"
      :on-before-close="beforeClose"
    >
      <p style="margin-bottom: 3px;">
        点击
        <Tag style="vertical-align: baseline;">
          确定
        </Tag>
        后
      </p>
      <p>经过 5 秒会自动关闭</p>
      <p>其他关闭途径将被阻断</p>
    </Modal>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const active = ref(false)
const title = ref('加载状态')
const loading = ref(false)

async function beforeClose(isConfirm: boolean) {
  if (isConfirm) {
    title.value = '5 秒后自动关闭'
    loading.value = true

    return new Promise(resolve => {
      setTimeout(() => {
        loading.value = false
        title.value = '加载状态'
        resolve(true)
      }, 5000)
    })
  }

  return false
}
</script>

<style>
.vxp-modal {
  min-width: 500px;
}
</style>
