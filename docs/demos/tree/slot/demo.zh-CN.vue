<template>
  <Tree :data="mockData">
    <template #node="{ data, expanded, toggleExpand }">
      <span
        :style="{
          color: expanded ? '#fa5252' : '#228be6',
          cursor: 'pointer'
        }"
        @click="toggleExpand()"
      >
        {{ data.label }}
      </span>
    </template>
  </Tree>
</template>

<script setup lang="ts">
const mockData = [
  {
    id: 1,
    label: '动物',
    parent: 0,
    expanded: true
  },
  {
    id: 2,
    label: '鸟类',
    parent: 5
  },
  {
    id: 3,
    label: '无脊椎动物',
    parent: 1
  },
  {
    id: 4,
    label: '哺乳动物',
    parent: 5
  },
  {
    id: 5,
    label: '脊椎动物',
    parent: 1
  },
  {
    id: 6,
    label: '喜鹊',
    parent: 2
  },
  {
    id: 7,
    label: '蚯蚓',
    parent: 3
  }
]
</script>
