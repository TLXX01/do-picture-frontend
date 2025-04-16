<template>
  <div id="userCenterPage">

    <a-card>

      <a-space class="left" direction="vertical">
        <a-card>
          <a-space>
            <a-avatar :size="64" :src="loginUser.userAvatar" />
            <a-space direction="vertical">
              <h2>{{ loginUser.userName }}</h2>
              <h4>ID: {{ loginUser.id }}</h4>
            </a-space>

          </a-space>

        </a-card>

        <a-card>
          <h2>动态</h2>
        </a-card>
      </a-space>

      <a-space class="right" direction="vertical">
        <a-card>
          <a-space direction="vertical">
            <a-button @click="doEdit">编辑资料</a-button>
            <a-button @click="doPassword">修改密码</a-button>
            <a-button @click="doAbout">关于鲸鱼</a-button>
            <a-button danger @click="doLogout">退出登录</a-button>
          </a-space>

        </a-card>

        <a-card >

        </a-card>
      </a-space>

    </a-card>
  </div>
</template>

<script lang="ts" setup>
//获取当前登录用户信息
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import {userLogoutUsingPost} from "@/api/userController.ts";
import {message} from "ant-design-vue";
import router from "@/router";
import ImageCropper from "@/components/ImageCropper.vue";

const loginUserStore = useLoginUserStore();
const loginUser = loginUserStore.loginUser;

const doEdit = () => {}
const doPassword = () => {}
const doAbout = () => {}
// 用户注销
const doLogout = async () => {
  const res = await userLogoutUsingPost()
  if (res.data.code === 0) {
    loginUserStore.setLoginUser({
      userName: '未登录',
    })
    message.success('退出登录成功')
    await router.push('/user/login')
  } else {
    message.error('退出登录失败，' + res.data.message)
  }
}
</script>

<style scoped>

#userCenterPage .right .cropper{
  width: 400px;
  height: 400px;
}

</style>
