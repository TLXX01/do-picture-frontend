<template>
  <div id="basicLayout">
    <GlobalLoading :show="isRouteLoading" />
    <a-layout style="min-height: 100vh">
      <a-layout-header class="header">
        <GlobalHeader />
      </a-layout-header>
      <a-layout>
        <GlobalSider class="sider" />
        <a-layout-content class="content">
          <router-view />
        </a-layout-content>
      </a-layout>
      <a-layout-footer class="footer">
        <a href="https://github.com/TLXX01?tab=repositories" target="_blank"> 鲸鱼 by TLXX01-YCLM </a>
      </a-layout-footer>
    </a-layout>
  </div>
</template>

<script setup lang="ts">
import GlobalHeader from '@/components/GlobalHeader.vue'
import GlobalSider from "@/components/GlobalSider.vue";
import GlobalLoading from "@/components/GlobalLoading.vue";
import {onBeforeUnmount, ref} from "vue";
import {useRouter} from "vue-router";

const router = useRouter()
const isRouteLoading = ref(false)
let loadingTimeout: number | null = null

// 路由切换时显示加载状态
router.beforeEach(() => {
  // 设置延迟显示，避免快速加载时的闪烁
  loadingTimeout = setTimeout(() => {
    isRouteLoading.value = true
  }, 200)
  return true
})

router.afterEach(() => {
  // 清除延时器并隐藏加载状态
  if (loadingTimeout) {
    clearTimeout(loadingTimeout)
  }
  isRouteLoading.value = false
})

// 组件销毁前清理
onBeforeUnmount(() => {
  if (loadingTimeout) {
    clearTimeout(loadingTimeout)
  }
})

</script>

<style scoped>
#basicLayout .header {
  padding-inline: 20px;
  background: white;
  color: unset;
  margin-bottom: 1px;
}

#basicLayout .sider {
  background: #fff;
  border-right: 0.5px solid #eee;
  padding-top: 20px;
}

#basicLayout :deep(.ant-menu-root) {
  border-bottom: none !important;
  border-inline-end: none !important;
}

#basicLayout .content {
  padding: 28px;
  background: linear-gradient(to right, #fefefe, #fff);
  margin-bottom: 28px;
}

#basicLayout .footer {
  background: #efefef;
  padding: 16px;
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  text-align: center;
}
</style>
