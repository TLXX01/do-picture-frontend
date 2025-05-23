<template>
  <a-modal
    class="avatar-cropper-modal"
    v-model:open="visible"
    title="编辑头像"
    :footer="false"
    @cancel="closeModal"
    :width="520"
  >
    <!-- 图片裁切组件 -->
    <div class="cropper-container">
      <VueCropper
        ref="cropperRef"
        :img="imageUrl"
        :autoCrop="true"
        :fixedBox="true"
        :centerBox="true"
        :canMoveBox="true"
        :info="true"
        outputType="png"
        :fixed="true"
        :fixedNumber="[1, 1]"
      />
    </div>

    <!-- 图片操作工具栏 -->
    <div class="cropper-actions">
      <a-space>
        <a-button class="action-button" @click="rotateLeft">
          <RotateLeftOutlined />
          <span>左旋转</span>
        </a-button>
        <a-button class="action-button" @click="rotateRight">
          <RotateRightOutlined />
          <span>右旋转</span>
        </a-button>
        <a-button class="action-button" @click="changeScale(1)">
          <FullscreenOutlined />
          <span>放大</span>
        </a-button>
        <a-button class="action-button" @click="changeScale(-1)">
          <FullscreenExitOutlined />
          <span>缩小</span>
        </a-button>
        <a-button type="primary" class="confirm-button" :loading="loading" @click="handleConfirm">
          <CheckSquareOutlined />
          <span>确认</span>
        </a-button>
      </a-space>
    </div>
  </a-modal>
</template>

<script lang="ts" setup>
import { ref, defineExpose } from 'vue'
import { message } from 'ant-design-vue'
import 'vue-cropper/dist/index.css'
import { VueCropper } from 'vue-cropper'
import {
  RotateLeftOutlined,
  RotateRightOutlined,
  FullscreenOutlined,
  FullscreenExitOutlined,
  CheckSquareOutlined,
} from '@ant-design/icons-vue'

// 注册 VueCropper 组件
// eslint-disable-next-line @typescript-eslint/no-unused-vars
const components = {
  VueCropper
}

interface Props {
  imageUrl?: string
  onSuccess?: (file: File) => void
}

const props = defineProps<Props>()
const visible = ref(false)
const loading = ref(false)
const cropperRef = ref()


// 打开弹窗
const openModal = () => {
  visible.value = true
}

// 关闭弹窗
const closeModal = () => {
  visible.value = false
}

// 向左旋转
const rotateLeft = () => {
  cropperRef.value?.rotateLeft()
}

// 向右旋转
const rotateRight = () => {
  cropperRef.value?.rotateRight()
}

// 缩放
const changeScale = (num: number) => {
  cropperRef.value?.changeScale(num)
}

// 确认裁切
const handleConfirm = () => {
  loading.value = true
  cropperRef.value?.getCropBlob((blob: Blob) => {
    try {
      // 检查文件大小（例如限制为5MB）
      if (blob.size > 5 * 1024 * 1024) {
        message.error('图片大小不能超过5MB')
        return
      }

      const file = new File([blob], 'avatar.png', { type: blob.type })
      props.onSuccess?.(file)
    } catch (error) {
      console.error('头像裁剪失败:', error)
      message.error('头像裁剪失败')
    } finally {
      loading.value = false
    }
  })
}

defineExpose({
  openModal,
  closeModal
})
</script>

<style scoped>
.avatar-cropper-modal {
  :deep(.ant-modal-content) {
    padding: 0;
    border-radius: 16px;
    overflow: hidden;
  }

  :deep(.ant-modal-header) {
    padding: 16px 24px;
    margin: 0;
    border-bottom: 1px solid #f0f0f0;
  }

  :deep(.ant-modal-body) {
    padding: 24px;
  }

  :deep(.ant-modal-close) {
    top: 16px;
  }
}

.cropper-container {
  background: #f8fafc;
  border-radius: 12px;
  overflow: hidden;
  margin-bottom: 16px;
}

.vue-cropper {
  height: 360px !important;
  background: #f8fafc;
}

.cropper-actions {
  text-align: center;
  padding-top: 8px;
}

.action-button {
  height: 36px;
  padding: 0 16px;
  border-radius: 18px;
  border-color: #e2e8f0;
  color: #64748b;
  transition: all 0.3s ease;
}

.action-button:hover {
  border-color: #ff8e53;
  color: #ff8e53;
}

.action-button .anticon {
  font-size: 16px;
}

.confirm-button {
  height: 36px;
  padding: 0 20px;
  border-radius: 18px;
  background: linear-gradient(135deg, #ff8e53 0%, #ff6b6b 100%);
  border: none;
  color: white;
  font-weight: 500;
  box-shadow: 0 4px 12px rgba(255, 107, 107, 0.2);
  transition: all 0.3s ease;
}

.confirm-button:not(:disabled):hover {
  transform: translateY(-1px);
  box-shadow: 0 6px 16px rgba(255, 107, 107, 0.3);
}

.confirm-button:not(:disabled):active {
  transform: translateY(1px);
}

/* 响应式调整 */
@media screen and (max-width: 768px) {
  .vue-cropper {
    height: 280px !important;
  }

  .action-button,
  .confirm-button {
    height: 32px;
    padding: 0 12px;
  }

  .action-button .anticon {
    font-size: 14px;
  }
}
</style>
