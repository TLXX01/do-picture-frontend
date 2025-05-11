<template>
  <div id="about">
    <a-modal title="关于鲸鱼" v-model:visible="visible" :footer="false" @cancel="closeModal">
      <p>鲸鱼是一款基于Vue3和TypeScript的图库系统，提供大量图片素材。</p>
      <p>作者：<a href="https://github.com/TLXX01?tab=repositories">TLXX01-YCLM</a></p>

    </a-modal>
  </div>
</template>
<script lang="ts" setup>
import {ref} from 'vue'

// 是否可见
const visible = ref(false)

// 打开弹窗
const openModal = () => {
  visible.value = true
}

// 关闭弹窗
const closeModal = () => {
  visible.value = false
}

// 暴露函数给父组件
defineExpose({
  openModal,
})

</script>
