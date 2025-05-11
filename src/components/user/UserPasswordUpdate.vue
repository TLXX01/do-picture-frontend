<template>
  <div>
    <a-modal title="修改密码" v-model:visible="visible" :footer="false" @cancel="closeModal">

      <a-form :model="userForm" layout="vertical" @finish="handleSubmit">

        <a-form-item
          label="旧密码"
          name="userPassword"
          :rules="[
          { required: true, message: '请输入旧密码' },
          { min: 8, message: '密码长度不能小于 8 位' },
        ]"
        >
          <a-input-password v-model:value="userForm.userPassword" />
        </a-form-item>

        <a-form-item
          label="新密码"
          name="newPassword"
          :rules="[
          { required: true, message: '请输入新密码' },
          { min: 8, message: '密码长度不能小于 8 位' },
        ]"
        >
          <a-input-password v-model:value="userForm.newPassword" />
        </a-form-item>

        <a-form-item
          label="确认密码"
          name="checkPassword"
          :rules="[
          { required: true, message: '请输入确认密码' },
          { min: 8, message: '确认密码长度不能小于 8 位' },
        ]"
        >
          <a-input-password v-model:value="userForm.checkPassword" />
        </a-form-item>

        <a-form-item style="text-align: center">
          <a-button type="primary" html-type="submit">修改</a-button>
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>
<script lang="ts" setup>
import {ref} from 'vue'
import {
  updatePasswordUsingPost
} from '@/api/userController.ts'
import { message } from 'ant-design-vue'

const userForm = ref<API.UpdatePasswordRequest>({})

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

// 更新用户密码
const handleSubmit = async () => {
  const res = await updatePasswordUsingPost({
    ...userForm.value,
  })
  if (res.data.code === 0 && res.data.data) {
    message.success('密码修改成功')
    closeModal()
  } else {
    message.error('密码修改失败' + res.data.message)
  }
}

</script>
