<template>
  <div id="personalInfoPage">
    <h2 class="title">个人信息</h2>
    <a-card class="info-card" bordered>
      <!-- 头像展示 -->
      <div class="avatar-container">
        <a-avatar :src="userInfo.userAvatar" size="large" icon="user" />
      </div>

      <!-- 个人信息表单（仅展示，不可编辑） -->
      <a-form :model="userInfo" name="personalInfo" layout="vertical" disabled>
        <a-form-item label="账号" name="userAccount">
          <a-input v-model:value="userInfo.userAccount" placeholder="暂无账号信息" />
        </a-form-item>

        <a-form-item label="用户昵称" name="userName">
          <a-input v-model:value="userInfo.userName" placeholder="暂无用户昵称" />
        </a-form-item>

        <a-form-item label="用户简介" name="userProfile">
          <a-textarea
            v-model:value="userInfo.userProfile"
            placeholder="暂无用户简介"
            :rows="4"
          />
        </a-form-item>
      </a-form>
    </a-card>
  </div>
</template>

<script setup lang="ts">
import { reactive } from 'vue'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'

// 定义用户信息数据结构，对应需求中的四个字段
interface UserInfo {
  userAccount: string
  userName: string
  userAvatar: string
  userProfile: string
}

// 初始化用户信息（优先从全局登录状态中获取，无数据则初始化空值）
const loginUserStore = useLoginUserStore()
const userInfo = reactive<UserInfo>({
  userAccount: loginUserStore.loginUser?.userAccount || '',
  userName: loginUserStore.loginUser?.userName || '',
  userAvatar: loginUserStore.loginUser?.userAvatar || '',
  userProfile: loginUserStore.loginUser?.userProfile || ''
})
</script>

<style scoped>
/* 沿用登录页面的布局风格，保持样式一致性 */
#personalInfoPage {
  max-width: 360px;
  margin: 0 auto;
}

.title {
  text-align: center;
  margin-bottom: 16px;
}

/* 头像容器样式 */
.avatar-container {
  display: flex;
  justify-content: center;
  margin-bottom: 24px;
}

/* 卡片样式优化，保持视觉统一 */
.info-card {
  padding: 24px;
}
</style>
