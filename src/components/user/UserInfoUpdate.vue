<template>
  <div>
    <a-modal title="编辑资料" v-model:visible="visible" :footer="false" @cancel="closeModal">

      <a-form :model="userForm" layout="vertical" @finish="handleSubmit">
        <a-form-item style="text-align: center">
          <UserAvatar :userData="userForm" :src="userForm.userAvatar" :size="64" :onSuccess="onAvatarSuccess"/>
        </a-form-item>

        <a-form-item label="昵称">
          <a-input v-model:value="userForm.userName" />
        </a-form-item>

        <a-form-item label="简介">
          <a-textarea v-model:value="userForm.userProfile" />
        </a-form-item>

        <a-form-item style="text-align: center">
          <a-button type="primary" html-type="submit">保存</a-button>
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>
<script lang="ts" setup>
import {onMounted, ref} from 'vue'
import {getUserVoByIdUsingGet, updateUserUsingPost} from '@/api/userController.ts'
import { message } from 'ant-design-vue'
import UserAvatar from "@/components/user/UserAvatar.vue";
import {useLoginUserStore} from "@/stores/useLoginUserStore.ts";

const loginUserStore = useLoginUserStore()
const loginUser = loginUserStore.loginUser

interface Props {
  user: API.UserVO
  onSuccess?: (updateUser: API.UserVO) => void
}
const props = defineProps<Props>()

const userForm = ref<API.UserUpdateRequest>({})

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

// 更新用户信息
const handleSubmit = async () => {
  const res = await updateUserUsingPost({
    id: props.user.id,
    ...userForm.value,
  })
  if (res.data.code === 0 && res.data.data) {
    message.success('信息修改成功')
    // 更新成功的图片信息传递给父组件

    props.onSuccess?.(userForm.value)
    closeModal()
  } else {
    message.error('信息修改失败' + res.data.message)
  }
}

//获取用户信息
const getUserInfo = async () => {
  // 获取到 id
  const id = props.user?.id
  if (id) {
    const res = await getUserVoByIdUsingGet({
      id,
    })
    if (res.data.code === 0 && res.data.data) {
      userForm.value = res.data.data
    }
  }
}

// 头像修改成功事件
const onAvatarSuccess = (userAvatar: string) => {
  userForm.value.userAvatar = userAvatar
  loginUser.userAvatar = userAvatar
}

onMounted(() => {
  getUserInfo()
})
</script>
