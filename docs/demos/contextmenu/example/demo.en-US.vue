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
import { User, Hammer, Fire } from '@vexip-ui/icons'

export default defineComponent({
  methods: {
    async contextmenu(event: MouseEvent) {
      // 未选择是则返回 null
      const selectedKey = await this.$contextmenu.open({
        clientX: event.clientX,
        clientY: event.clientY,
        appear: true,
        configs: [
          {
            key: 'Option 1'
          },
          {
            key: 'Option 2',
            icon: User,
            shortcut: 'Ctrl+A'
          },
          {
            key: 'Option 3',
            icon: Hammer,
            divided: true,
            iconColor: 'cyan',
            children: [
              {
                key: 'Option 3-1',
                disabled: true
              },
              {
                key: 'Option 3-2',
                divided: true
              },
              {
                key: 'Option 3-3',
                children: [
                  {
                    key: 'Option 3-3-1',
                    shortcut: 'Ctrl+L'
                  },
                  {
                    key: 'Option 3-3-2'
                  }
                ]
              }
            ]
          },
          {
            key: 'Option 4',
            shortcut: 'Alt+I',
            color: 'red'
          },
          {
            key: 'Option 5',
            shortcut: 'Ctrl+Alt+I',
            icon: Fire,
            color: 'blue',
            iconColor: 'green'
          }
        ]
      })

      console.info(selectedKey)
    }
  }
})
</script>
