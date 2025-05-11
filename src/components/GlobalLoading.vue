<template>
  <div class="global-loading" v-if="show">
    <div class="loading-content">
      <a-spin :indicator="indicator" />
      <div class="loading-text">
        <h3>加载中</h3>
        <p>请稍候片刻...</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { h } from 'vue'
import { LoadingOutlined } from '@ant-design/icons-vue'

defineProps<{
  show: boolean
}>()

const indicator = h(LoadingOutlined, {
  style: {
    fontSize: '24px',
    color: '#ff8e53',
  },
})
</script>

<style scoped>
.global-loading {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(4px);
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  animation: fadeIn 0.3s ease;
}

.loading-content {
  text-align: center;
  padding: 32px;
  border-radius: 12px;
  background: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.loading-text {
  margin-top: 16px;

  h3 {
    color: #1a1a1a;
    font-size: 16px;
    margin: 0 0 4px;
  }

  p {
    color: #64748b;
    font-size: 14px;
    margin: 0;
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
