<template>
  <Select v-model:value="value" :prefix="User" :options="options"></Select>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { User } from '@vexip-ui/icons'

const value = ref('')
const options = ['Option 1', 'Option 2', 'Option 3']
</script>

<style>
.vxp-select {
  max-width: 400px;
}
</style>
