<template>
  <Tabs v-model:active="active">
    <TabPane label="标签页1">
      <p>标签页一的内容。</p>
    </TabPane>
    <TabPane label="标签页2">
      <p>标签页二的内容。</p>
    </TabPane>
    <TabPane label="标签页3">
      <p>标签页三的内容。</p>
    </TabPane>
  </Tabs>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const active = ref('标签页1')
</script>

<style>
.vxp-tabs {
  max-width: 500px;
}
</style>
