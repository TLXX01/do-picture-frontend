<template>
  <div id="userCenterPage">
    <div class="userCenter-container">
      <!-- 流星背景 -->
      <div class="meteor-background">
        <div class="meteor meteor-1"></div>
        <div class="meteor meteor-2"></div>
        <div class="meteor meteor-3"></div>
        <div class="meteor meteor-4"></div>
        <div class="meteor meteor-5"></div>
      </div>
      <div class="main-content">

        <div class="content-layout">
          <!-- 左侧区域：用户信息和成长足迹 -->
          <div class="left-section">
            <!-- 用户信息区域 -->
            <div class="user-info-container">
              <div class="avatar-and-text">
                <div class="avatar-container">
                  <a-avatar
                    class="user-avatar"
                    :src="loginUser.userAvatar || getDefaultAvatar(loginUser.userName)"
                    :size="80"
                  />
                </div>
                <div class="text-info-container">
                  <div class="user-name">{{ loginUser.userName }}</div>
                  <div class="user-id">ID: {{ loginUser.id }}</div>
                </div>
              </div>
            </div>
            <!-- 签到日历 -->
            <div class="sign-in-calendar">
              <div class="calendar-header">
                <h3>成长足迹</h3>
                <a-select
                  v-model:value="selectedYear"
                  :options="yearOptions"
                  class="year-selector"
                  @change="fetchSignInData"
                />
              </div>
              <v-chart :option="calendarOption" autoresize class="calendar-chart" />
            </div>
          </div>


          <!-- 右侧区域：用户操作选项 -->
          <div class="right-section">
            <!-- 按钮 -->
            <div class="button-container">
              <a-button class="custom-button" @click="doUpdate">
                <span class="button-content">
                  <EditOutlined class="button-icon edit-icon" />
                  <span class="button-text">编辑资料</span>
                </span>
                <RightOutlined class="arrow-icon" />
              </a-button>

              <a-button class="custom-button" @click="doPassword">
                <span class="button-content">
                  <LockOutlined class="button-icon password-icon" />
                  <span class="button-text">修改密码</span>
                </span>
                <RightOutlined class="arrow-icon" />
              </a-button>

              <a-button class="custom-button" @click="doAbout">
                <span class="button-content">
                  <InfoCircleOutlined class="button-icon about-icon" />
                  <span class="button-text">关于鲸鱼</span>
                </span>
                <RightOutlined class="arrow-icon" />
              </a-button>

              <a-button class="custom-button" @click="doLogout">
                <span class="button-content">
                  <LogoutOutlined class="button-icon about-icon" />
                  <span class="button-text">退出登录</span>
                </span>
                <RightOutlined class="arrow-icon" />
              </a-button>
            </div>

            <!-- 插画区域 -->
            <div class="illustration-container">
              <div class="illustration-content">
                <div ref="animationContainer" class="animation-container"></div>
                <p class="illustration-text">奥利奥利奥利给！！！！！</p>
              </div>
            </div>
          </div>
        </div>

        <UserInfoUpdate ref="userInfoUpdateRef" :user="oldUser" :onSuccess="onUserSuccess" />

        <UserPasswordUpdate ref="userPasswordRef" />

        <About ref="aboutRef" />

      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
//获取当前登录用户信息
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import {
  addUserSignInUsingPost, getUserSignInRecordUsingGet,
  userLogoutUsingPost
} from '@/api/userController.ts'
import {
  EditOutlined,
  LockOutlined,
  InfoCircleOutlined,
  LogoutOutlined,
  RightOutlined
} from '@ant-design/icons-vue'
import { message } from 'ant-design-vue'
import router from '@/router'
import UserInfoUpdate from '@/components/user/UserInfoUpdate.vue'
import About from '@/components/About.vue'
import {computed, onMounted, ref, type UnwrapRef} from 'vue'
import VChart from "vue-echarts";
import * as echarts from "echarts";
import UserPasswordUpdate from "@/components/user/UserPasswordUpdate.vue";

const loginUserStore = useLoginUserStore()
const loginUser = loginUserStore.loginUser

const oldUser = loginUser

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

const userInfoUpdateRef = ref()

const aboutRef = ref()

const userPasswordRef = ref()

// 修改用户信息
const doUpdate = async () => {
  userInfoUpdateRef.value?.openModal()
}

// 修改成功事件
const onUserSuccess = (newUser: API.UserVO) => {
  loginUser.id = newUser.id
  loginUser.userName = newUser.userName
}

const doPassword = () => {
  userPasswordRef.value?.openModal()
}

const doAbout = () => {
  aboutRef.value?.openModal()
}

// 获取默认头像
const getDefaultAvatar = (userName: UnwrapRef<API.LoginUserVO['userName']> | undefined) => {
  const defaultName = userName || 'Guest'
  return `https://api.dicebear.com/7.x/adventurer/svg?seed=${encodeURIComponent(defaultName)}&backgroundColor=ffd5dc,ffdfbf,ffd5dc`
}

// 添加新的响应式变量
const selectedYear = ref(new Date().getFullYear())
const signInData = ref<number[]>([])

// 生成年份选项
const yearOptions = computed(() => {
  const currentYear = new Date().getFullYear()
  return Array.from({ length: 3 }, (_, i) => ({
    label: `${currentYear - i}年`,
    value: currentYear - i
  }))
})


// 获取签到数据
const fetchSignInData = async () => {
  try {
    const res = await getUserSignInRecordUsingGet({
      year: selectedYear.value || new Date().getFullYear()
    })
    if (res.data.code === 0) {
      signInData.value = res.data.data || []
    }
  } catch (error) {
    console.error('获取签到记录失败:', error)
  }
}

// 日历图配置
const calendarOption = computed(() => {
  // 根据屏幕宽度判断是否为移动端
  const isMobile = window.innerWidth <= 768

  return {
    tooltip: {
      formatter: (params: any) => {
        return `${params.value[0]}<br/>${params.value[1]? '✨ 浅浅地踩了一下' : '没有留下足迹'}`
      },
      backgroundColor: 'rgba(255, 255, 255, 0.95)',
      borderColor: '#e2e8f0',
      textStyle: {
        color: '#64748b',
        fontSize: 12
      },
      padding: [8, 12],
      borderRadius: 8
    },
    visualMap: {
      min: 0,
      max: 2,
      calculable: false,
      orient: 'horizontal',
      left: 'center',
      bottom: 0,
      showLabel: false,
      show: false,
      inRange: {
        color: ['#f1f5f9', '#ffd5dc', '#ff8e53']
      }
    },
    calendar: {
      top: 20,
      left: 30,
      right: 30,
      cellSize: ['auto', 20],
      range: selectedYear.value,
      itemStyle: {
        borderWidth: 2,
        borderColor: '#fff',
        borderRadius: 2
      },
      yearLabel: { show: false },
      dayLabel: {
        firstDay: 1,
        nameMap: ['日', '一', '二', '三', '四', '五', '六'],
        color: '#94a3b8',
        fontSize: 12
      },
      monthLabel: {
        nameMap: isMobile
          ? ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12']
          : ['1月', '2月', '3月', '4月', '5月', '6月', '7月', '8月', '9月', '10月', '11月', '12月'],
        color: '#64748b',
        fontSize: 12,
        align: 'left'
      }
    },
    series: {
      type: 'heatmap',
      coordinateSystem: 'calendar',
      data: signInData.value.map(day => {
        const date = new Date(selectedYear.value, 0, day)
        return [echarts.format.formatTime('yyyy-MM-dd', date), 1]
      })
    }
  }
})

// 添加自动签到函数
const autoSignIn = async () => {
  if (!loginUserStore.loginUser.id) {
    return
  }

  const res = await addUserSignInUsingPost()
  if (res.data.code === 0) {
    // 签到成功后更新签到数据
    await fetchSignInData()

  }
}

// 在 onMounted 钩子中调用自动签到
onMounted(async () => {
  // 确保用户已登录
  if (loginUserStore.loginUser.id) {
    await autoSignIn()
    await fetchSignInData()// 刷新签到数据显示
  }
})
</script>

<style scoped>
#userCenterPage {
  min-height: calc(100vh - 120px);
  background: #f8fafc;
  position: relative;
  overflow: hidden;
  width: 100%;
  display: flex;
  justify-content: center;
}

/* 设置界面容器 */
.userCenter-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 24px 20px;
  height: 100%;
  position: relative;
}

/* 流星背景样式 */
.meteor-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 0;
}

.meteor {
  position: absolute;
  width: 2px;
  height: 2px;
  background: linear-gradient(45deg, #ff8e53, #ff6b6b);
  border-radius: 50%;
  animation: meteor-fall linear infinite;
  opacity: 0;
}

.meteor::before {
  content: '';
  position: absolute;
  width: 100px;
  height: 1px;
  transform: translateX(-100%);
  background: linear-gradient(90deg, #ff8e53, transparent);
}

.meteor-1 {
  top: -10%;
  left: 20%;
  animation-duration: 6s;
  animation-delay: 0s;
}

.meteor-2 {
  top: -10%;
  left: 40%;
  animation-duration: 8s;
  animation-delay: 2s;
}

.meteor-3 {
  top: -10%;
  left: 60%;
  animation-duration: 7s;
  animation-delay: 4s;
}

.meteor-4 {
  top: -10%;
  left: 80%;
  animation-duration: 9s;
  animation-delay: 1s;
}

.meteor-5 {
  top: -10%;
  left: 30%;
  animation-duration: 5s;
  animation-delay: 3s;
}

@keyframes meteor-fall {
  0% {
    transform: translate(0, 0) rotate(45deg);
    opacity: 1;
  }
  20% {
    opacity: 1;
  }
  60% {
    opacity: 0;
  }
  100% {
    transform: translate(500px, 500px) rotate(45deg);
    opacity: 0;
  }
}

.main-content {
  position: relative;
  z-index: 1;
  width: 100%;
}

/* 左右布局容器 */
.content-layout {
  display: flex;
  gap: 20px;
  padding-top: 24px;
  width: 100%;
  height: 100%;
}

/* 左侧区域 */
.left-section {
  flex: 1;
  min-width: 800px;
  width: calc(100% - 360px);
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* 右侧区域 */
.right-section {
  width: 320px;
  display: flex;
  flex-direction: column;
  gap: 16px;
  flex-shrink: 0;
}

/* 用户信息区域 */
.user-info-container {
  background: white;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  height: 112px;
  display: flex;
  align-items: center;
  background: linear-gradient(135deg, rgba(255, 142, 83, 0.05) 0%, rgba(255, 107, 107, 0.05) 100%);
}

.avatar-and-text {
  display: flex;
  align-items: center;
  gap: 16px;
}

.avatar-container {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.user-avatar {
  border-radius: 16px;
  border: 2px solid white;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  background-color: #fff6f3;
}

.text-info-container {
  flex: 1;
}

.user-name {
  font-size: 18px;
  font-weight: 600;
  color: #1a1a1a;
  margin-bottom: 4px;
}

.user-id {
  font-size: 13px;
  color: #94a3b8;
}

/* 按钮容器 */
.button-container {
  background: white;
  border-radius: 16px;
  padding: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  height: 240px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  background: linear-gradient(135deg, rgba(255, 142, 83, 0.05) 0%, rgba(255, 107, 107, 0.05) 100%);
}

.custom-button{
  margin-bottom: 8px;
}

/* 按钮样式 */
:deep(.custom-button) {
  width: 100%;
  height: 52px;
  padding: 0 16px;
  border: none;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-radius: 12px;
  margin-bottom: 8px !important;
  transition: all 0.3s ease;
}

/* 按钮背景色 */
:deep(.custom-button:has(.edit-icon)) {
  background: rgba(14, 165, 233, 0.05);
}

:deep(.custom-button:has(.password-icon)) {
  background: rgba(139, 92, 246, 0.05);
}

:deep(.custom-button:has(.about-icon)) {
  background: rgba(245, 158, 11, 0.05);
}

:deep(.custom-button:has(.destroy-icon)) {
  background: rgba(239, 68, 68, 0.05);
}

/* 按钮悬停效果 */
:deep(.custom-button:has(.edit-icon):hover) {
  background: rgba(14, 165, 233, 0.1);
}

:deep(.custom-button:has(.password-icon):hover) {
  background: rgba(139, 92, 246, 0.1);
}

:deep(.custom-button:has(.about-icon):hover) {
  background: rgba(245, 158, 11, 0.1);
}

:deep(.custom-button:has(.destroy-icon):hover) {
  background: rgba(239, 68, 68, 0.1);
}

.button-content {
  display: flex;
  align-items: center;
  gap: 12px;
}

/* 图标颜色 */
:deep(.edit-icon) {
  color: #0ea5e9;
}
:deep(.password-icon) {
  color: #8b5cf6;
}
:deep(.about-icon) {
  color: #f59e0b;
}
:deep(.destroy-icon) {
  color: #ef4444;
}

:deep(.button-text) {
  font-size: 15px;
  color: #1a1a1a;
}

:deep(.arrow-icon) {
  color: #94a3b8;
  font-size: 14px;
}

/* 模态框样式优化 */
:deep(.ant-modal) {
  border-radius: 20px;
  overflow: hidden;
}

:deep(.ant-modal-content) {
  padding: 24px;
}

:deep(.ant-modal-header) {
  border-bottom: none;
  padding: 0 0 20px 0;
}

:deep(.ant-modal-title) {
  font-size: 18px;
  font-weight: 600;
  color: #1a1a1a;
}

:deep(.ant-modal-body) {
  padding: 0;
}

/* 头像上传区域优化 */
.avatar-upload-container {
  position: relative;
  text-align: center;
  padding: 32px 0;
  margin-bottom: 24px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.avatar-wrapper {
  position: relative;
  display: inline-block;
  padding: 4px;
  background: white;
  border-radius: 50%;
  cursor: pointer;
  margin: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  width: fit-content;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
  background: linear-gradient(135deg, rgba(255, 142, 83, 0.1) 0%, rgba(255, 107, 107, 0.1) 100%);

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 16px rgba(255, 142, 83, 0.15);
  }

  &:active {
    transform: translateY(0);
  }
}

:deep(.ant-avatar) {
  border-radius: 50%;
  border: 2px solid white;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  width: 80px !important;
  height: 80px !important;
  background-color: #fff6f3;

  &:hover {
    border-color: #ff8e53;
  }
}

.upload-icon {
  position: absolute;
  bottom: -4px;
  right: -4px;
  width: 28px;
  height: 28px;
  background: linear-gradient(135deg, #ff8e53 0%, #ff6b6b 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(255, 107, 107, 0.3);
  z-index: 2;
  border: 2px solid white;
}

/* 确保上传图标可以被点击 */
.upload-icon:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(255, 107, 107, 0.4);
}

/* 响应式调整 */
@media screen and (max-width: 768px) {
  .avatar-upload-container {
    padding: 24px 0;
    margin-bottom: 20px;
    width: 100%;
  }

  .avatar-wrapper {
    padding: 3px;
    margin: 0 auto;
  }

  :deep(.ant-avatar) {
    width: 72px !important;
    height: 72px !important;
  }

  .upload-icon {
    width: 24px;
    height: 24px;
    bottom: -3px;
    right: -3px;
  }
}

/* 添加加载状态的样式 */
.uploading {
  opacity: 0.7;
  pointer-events: none;
}

/* 提交按钮样式 */
:deep(.submit-button) {
  background: linear-gradient(135deg, #ff8e53 0%, #ff6b6b 100%);
  border: none;
  color: white;
  width: 200px;
  height: 44px;
  border-radius: 22px;
  font-size: 15px;
  font-weight: 500;
  box-shadow: 0 4px 12px rgba(255, 107, 107, 0.2);
  transition: all 0.3s ease;
}

:deep(.submit-button:hover) {
  transform: translateY(-1px);
  box-shadow: 0 6px 16px rgba(255, 107, 107, 0.3);
}

:deep(.submit-button:active) {
  transform: translateY(1px);
}

/* 表单样式优化 */
.form-container {
  padding: 0;
}

:deep(.ant-form-item) {
  margin-bottom: 20px;
}

:deep(.ant-form-item-label) {
  padding-bottom: 4px;
}

:deep(.ant-form-item-label > label) {
  font-size: 14px;
  color: #64748b;
}

:deep(.ant-input) {
  border-radius: 10px;
  border-color: #e2e8f0;
  padding: 8px 12px;
  transition: all 0.3s ease;
}

:deep(.ant-input:hover) {
  border-color: #ff8e53;
}

:deep(.ant-input:focus) {
  border-color: #ff8e53;
  box-shadow: 0 0 0 2px rgba(255, 142, 83, 0.1);
}

:deep(.ant-input[disabled]) {
  background: #f8fafc;
  color: #94a3b8;
  border-color: #e2e8f0;
}

/* 修改密码表单样式 */
.form-item-wrapper {
  margin-bottom: 16px;
}

.form-label {
  font-size: 14px;
  color: #64748b;
  margin-bottom: 4px;
  display: block;
}

:deep(.form-input) {
  border-radius: 10px;
  border-color: #e2e8f0;
}

/* 模态框按钮样式 */
:deep(.ant-modal-footer) {
  text-align: center;
  border-top: none;
  padding: 24px 0 0 0;
}

:deep(.ant-modal-footer .ant-btn-primary) {
  background: linear-gradient(135deg, #ff8e53 0%, #ff6b6b 100%);
  border: none;
  width: 200px;
  height: 44px;
  border-radius: 22px;
  font-size: 15px;
  font-weight: 500;
  box-shadow: 0 4px 12px rgba(255, 107, 107, 0.2);
  transition: all 0.3s ease;
}

:deep(.ant-modal-footer .ant-btn-primary:hover) {
  transform: translateY(-1px);
  box-shadow: 0 6px 16px rgba(255, 107, 107, 0.3);
}

:deep(.ant-modal-footer .ant-btn-primary:active) {
  transform: translateY(1px);
}

/* 关于我们模态框样式 */
:deep(.ant-modal.about-modal .ant-modal-body) {
  text-align: center;
  padding: 32px 0;
}

:deep(.ant-modal.about-modal p) {
  font-size: 15px;
  color: #1a1a1a;
  margin-bottom: 8px;
}

:deep(.ant-modal.about-modal a) {
  color: #94a3b8;
  text-decoration: none;
  font-size: 13px;
  transition: color 0.3s ease;
}

:deep(.ant-modal.about-modal a:hover) {
  color: #ff8e53;
}

/* 确认注销模态框样式 */
:deep(.ant-modal.confirm-modal .ant-modal-body) {
  padding: 32px 24px;
  text-align: center;
  font-size: 15px;
  color: #1a1a1a;
}

/* 响应式调整 */
@media screen and (max-width: 768px) {
  .content-layout {
    flex-direction: column;
    padding-top: 12px;
    gap: 12px;
  }

  .left-section,
  .right-section {
    width: 100%;
    max-width: none;
    gap: 12px;
    min-width: unset;
  }

  #UserCenterPage {
    padding: 12px 0;
    display: block;
    min-height: calc(100vh - 160px);
  }

  .setting-container {
    padding: 0;
  }

  .main-content {
    padding: 0 4px;
  }

  .user-info-container,
  .button-container {
    border-radius: 12px;
    margin: 0;
    box-shadow: none;
  }

  .user-info-container {
    margin-bottom: 12px;
  }

  .sign-in-calendar {
    padding: 16px 12px;
    margin: 12px 0;
    border-radius: 12px;
    min-height: 280px;
    width: calc(100% - 8px);
    margin: 4px auto;
  }

  .calendar-chart {
    height: auto;
    margin-top: 4px;
    min-height: 200px;
    width: 100% !important;
  }

  .user-info-container {
    padding: 16px;
  }

  .button-container {
    padding: 12px;
  }

  /* 在移动端隐藏流星效果 */
  .meteor-background {
    display: none;
  }
}

/* 修改密码模态框样式 */
.password-form {
  padding: 0 12px;
}

:deep(.password-modal .ant-form-item-label > label) {
  font-size: 14px;
  color: #64748b;
}

:deep(.password-modal .ant-form-item-label > label.ant-form-item-required::before) {
  color: #ff6b6b;
}

:deep(.password-modal .ant-input) {
  height: 42px;
  border-radius: 10px;
  border-color: #e2e8f0;
  transition: all 0.3s ease;
}

:deep(.password-modal .ant-input:hover) {
  border-color: #ff8e53;
}

:deep(.password-modal .ant-input:focus) {
  border-color: #ff8e53;
  box-shadow: 0 0 0 2px rgba(255, 142, 83, 0.1);
}







@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.1);
    opacity: 0.8;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

/* 响应式调整 */
@media screen and (max-width: 768px) {
  .logout-modal-content {
    padding: 24px 16px;
  }

  .warning-icon {
    font-size: 40px;
  }

  .modal-title {
    font-size: 16px;
  }

  .modal-desc {
    font-size: 13px;
    padding: 0 8px;
  }

  .modal-actions {
    gap: 8px;
  }

  .cancel-button,
  .confirm-button {
    min-width: 90px;
    height: 36px;
    font-size: 13px;
  }
}

/* 签到日历样式 */
.sign-in-calendar {
  background: white;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  height: calc(100% - 128px);
  display: flex;
  flex-direction: column;
  margin-top: 16px;
  width: 100%;
  background: linear-gradient(135deg, rgba(14, 165, 233, 0.05) 0%, rgba(139, 92, 246, 0.05) 100%);
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.calendar-header h3 {
  font-size: 16px;
  font-weight: 600;
  color: #1a1a1a;
  margin: 0;
}

.year-selector {
  width: 120px;
}

.calendar-chart {
  height: 200px;
  width: 100% !important;
  margin-top: 8px;
  flex: 1;
}

/* 下拉选择器样式 */
:deep(.ant-select-selector) {
  border-radius: 8px !important;
  border-color: #e2e8f0 !important;
  height: 32px !important;
  line-height: 32px !important;
}

:deep(.ant-select-selection-item) {
  line-height: 30px !important;
  font-size: 13px;
  color: #64748b;
}

/* 响应式调整 */
@media screen and (max-width: 768px) {
  .sign-in-calendar {
    padding: 16px 12px;
    margin: 12px 0;
    border-radius: 12px;
    min-height: 280px;
    width: calc(100% - 8px);
    margin: 4px auto;
  }

  .calendar-header {
    margin-bottom: 12px;
  }

  .calendar-header h3 {
    font-size: 15px;
    color: #334155;
  }

  .year-selector {
    width: 90px;
  }

  .calendar-chart {
    height: auto;
    margin-top: 4px;
    min-height: 200px;
    width: 100% !important;
  }

  /* 优化日历在移动端的显示 */
  :deep(.calendar-chart) {
    .echarts-tooltip {
      padding: 4px 8px !important;
      border-radius: 6px !important;
      font-size: 12px !important;
    }

    .calendar-heatmap text {
      font-size: 11px !important;
    }
  }
}

/* 插画容器样式 */
.illustration-container {
  background: white;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  height: calc(100% - 256px);
  text-align: center;
  display: none;
  display: flex;
  flex-direction: column;
  justify-content: center;
  background: linear-gradient(135deg, rgba(139, 92, 246, 0.05) 0%, rgba(14, 165, 233, 0.05) 100%);
}

.illustration-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.animation-container {
  width: 160px;
  height: 160px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: center;
}

.illustration-text {
  font-size: 14px;
  color: #64748b;
  margin: 0;
  margin-top: 8px;
}

/* 仅在PC端显示插画 */
@media screen and (min-width: 769px) {
  .illustration-container {
    display: block;
  }

  .button-container {
    flex: 0 0 auto;
  }
}

/* 移动端隐藏插画 */
@media screen and (max-width: 768px) {
  .illustration-container {
    display: none;
  }
}


/* 移动端适配 */
@media screen and (max-width: 768px) {
  .user-info-container {
    padding: 20px 16px;
  }

  .user-meta {
    margin-top: 6px;
  }
}

.forgot-password-link {
  text-align: center;
  margin-top: -12px;
  margin-bottom: 16px;
  font-size: 14px;
  color: #64748b;
}

.forgot-password-link a {
  color: #ff8e53;
  margin-left: 4px;
  cursor: pointer;
  transition: color 0.3s ease;
}

.forgot-password-link a:hover {
  color: #ff7a3d;
}

.verify-code-container {
  display: flex;
  gap: 12px;
}

.verify-code-container .ant-input {
  flex: 1;
}





</style>
