<template>
  <div
    style="width: 400px; height: 100px; border: 1px solid #ccc;"
    @contextmenu.prevent="contextmenu"
  >
    在此处右键
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { Contextmenu } from 'vexip-ui'

export default defineComponent({
  setup() {
    async function contextmenu(event: MouseEvent) {
      // 未选择是则返回 null
      const selectedKey = await Contextmenu.open({
        clientX: event.clientX,
        clientY: event.clientY,
        appear: true,
        configs: [
          { key: '选项1' },
          { key: '选项2' },
          {
            key: '选项3',
            children: [{ key: '选项3-1' }, { key: '选项3-2' }, { key: '选项3-3' }]
          },
          { key: '选项4' }
        ]
      })

      console.info(selectedKey)
    }

    return { contextmenu }
  }
})
</script>
