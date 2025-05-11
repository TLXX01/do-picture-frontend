<template>
  <div class="avatar-upload">
    <div class="avatar-wrapper">
      <a-avatar
        :src="userData?.userAvatar"
        :size="80"
      />
      <div class="upload-icon" @click="showFileUploadDialog">
        <PlusOutlined />
      </div>
    </div>
    <input
      type="file"
      ref="fileInput"
      style="display: none"
      accept="image/*"
      @change="handleAvatarUpload"
    />

    <!-- 头像裁剪组件 -->
    <AvatarCropper
      ref="avatarCropperRef"
      :imageUrl="tempImageUrl"
      @success="handleCroppedAvatar"
    />

  </div>
</template>
<script lang="ts" setup>
import { ref } from 'vue'
import type { UploadProps } from 'ant-design-vue'
import { message } from 'ant-design-vue'
import AvatarCropper from "@/components/user/AvatarCropper.vue";
import {uploadAvatarUsingPost} from "@/api/userController.ts";
import {
  PlusOutlined
} from '@ant-design/icons-vue'

interface Props {
  userData?: API.UserVO
  src?: string
  onSuccess?: (userAvatar: string) => void
}

const props = defineProps<Props>()

// 修改文件输入框的引用名称
const fileInput = ref<HTMLInputElement | null>(null)

// 修改显示文件选择对话框的方法
const showFileUploadDialog = () => {
  // 确保DOM元素已经挂载
  if (fileInput.value) {
    fileInput.value.click()
  } else {
    console.error('文件输入框未找到')
  }
}

const avatarCropperRef = ref()
// 临时图片 URL
const tempImageUrl = ref('')


// 修改头像上传处理方法
const handleAvatarUpload = (e: Event) => {
  const file = (e.target as HTMLInputElement).files?.[0]
  if (file) {
    // 创建临时 URL 并打开裁剪器
    tempImageUrl.value = URL.createObjectURL(file)
    if (avatarCropperRef.value) {
      avatarCropperRef.value.openModal()
    }
    // 清空 input 值，允许重复选择同一文件
    (e.target as HTMLInputElement).value = ''
  }
}

// 处理裁剪后的头像
const handleCroppedAvatar = async (file: File) => {
  try {
    const params = {
      id:props.userData?.id
    }
    const res = await uploadAvatarUsingPost(params, {}, file, {})
    if (res.data.code === 0) {
      message.success('头像上传成功')

      // 上传成功后关闭裁剪框并将头像传递给父组件
      props.onSuccess?.(res.data.data)


      if (avatarCropperRef.value) {
        avatarCropperRef.value.closeModal()
      }
    }
  } catch (error) {
    console.error('头像上传失败:', error)
    message.error('头像上传失败')
  }
}

</script>
<style scoped>

</style>
