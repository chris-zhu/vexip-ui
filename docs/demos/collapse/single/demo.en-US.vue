<template>
  <div>
    <CollapsePane title="独立面板一" arrow-type="left">
      <p>Some content to fill</p>
      <p>Some content to fill</p>
    </CollapsePane>
    <br />
    <CollapsePane card>
      <template #title>
        <Icon><ChartLine></ChartLine></Icon>
        独立面板二
      </template>
      <p>Some content to fill</p>
      <p>Some content to fill</p>
    </CollapsePane>
  </div>
</template>

<script setup lang="ts">
import { ChartLine } from '@vexip-ui/icons'
</script>
