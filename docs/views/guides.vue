<template>
  <Row tag="section" :class="prefix">
    <Column tag="aside" :class="`${prefix}__sider`" flex="300px">
      <Scroll use-y-bar height="100%" :delta-y="60">
        <Menu
          v-model:active="currentMenu"
          marker-type="left"
          :class="`${prefix}__menu`"
          @select="selectGuide"
        >
          <MenuItem v-for="menu in menus" :key="menu.label" :label="menu.label">
            {{ $t(`guides.${menu.i18n}`) }}
          </MenuItem>
        </Menu>
      </Scroll>
    </Column>
    <Column tag="section" :class="`${prefix}__content`" style="flex: 1 1 calc(100% - 300px);">
      <NativeScroll
        ref="scroll"
        appear
        use-y-bar
        height="100%"
        :delta-y="50"
      >
        <main :class="`${prefix}__main`">
          <router-view v-slot="{ Component }">
            <transition name="vxp-fade" mode="out-in">
              <component :is="Component"></component>
            </transition>
          </router-view>
        </main>
        <Footer></Footer>
      </NativeScroll>
      <section id="toc-anchor" class="toc-anchor"></section>
    </Column>
  </Row>
</template>

<script setup lang="ts">
import { ref, watch, inject } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import Footer from '../common/footer.vue'
import { getGuideConfig } from '../router/guides'

// eslint-disable-next-line @typescript-eslint/consistent-type-imports
import type { Scroll } from 'vexip-ui'

const globalState = inject('globalState', { language: __ROLLBACK_LANG__ })

const prefix = 'guides'

const currentMenu = ref('')
const menus = getGuideConfig()

const scroll = ref<InstanceType<typeof Scroll> | null>(null)

const router = useRouter()
const route = useRoute()

watch(
  () => route.path,
  value => {
    if (value.startsWith(`/${globalState.language}/guides`)) {
      currentMenu.value = value.split('/')[3]
    }

    if (!currentMenu.value) {
      currentMenu.value = menus[0].label
    }

    scroll.value?.scrollTo(0, 0)
  },
  { immediate: true }
)

function selectGuide(label: string) {
  router.push(`/${globalState.language}/guides/${label}`)
}
</script>

<style lang="scss">
.guides {
  &,
  &__sider,
  &__content {
    position: relative;
    height: 100%;
  }

  &__sider {
    border-right: var(--vxp-border-light-2);
    transition: var(--vxp-transition-border);
  }

  &__menu {
    padding: 40px 0 20px;
    border-right: 0;
  }

  &__main {
    height: 100%;
  }

  .toc-anchor {
    position: absolute;
    top: 40px;
    right: 13.5em;
    width: 12.5em;
    transform: translateX(100%);

    .vxp-anchor {
      width: 100%;
      font-size: 12px;

      &__link {
        width: 100%;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }
    }
  }
}
</style>
