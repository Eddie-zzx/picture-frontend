<template>
  <div id="personalInfoPage">
    <h2 class="title">个人信息</h2>
    <a-card class="info-card" bordered :class="{ 'edit-mode': isEditing }">
      <!-- 头像展示 -->
      <div class="avatar-container">
        <a-avatar :src="userInfo.userAvatar" size="large" icon="user" />
      </div>

      <!-- 个人信息表单 -->
      <a-form :model="userInfo" name="personalInfo" layout="vertical">
        <a-form-item label="账号" name="userAccount">
          <a-input v-model:value="userInfo.userAccount" placeholder="暂无账号信息" disabled />
        </a-form-item>

        <a-form-item label="用户昵称" name="userName">
          <a-input
            v-model:value="userInfo.userName"
            placeholder="暂无用户昵称"
            :disabled="!isEditing"
          />
        </a-form-item>

        <a-form-item label="用户简介" name="userProfile">
          <a-textarea
            v-model:value="userInfo.userProfile"
            placeholder="暂无用户简介"
            :rows="4"
            :disabled="!isEditing"
          />
        </a-form-item>
      </a-form>

      <!-- 按钮容器 -->
      <div class="button-container">
        <!-- 非编辑状态 -->
        <a-button v-if="!isEditing" type="primary" @click="startEditing"> 编辑 </a-button>

        <!-- 编辑状态 -->
        <template v-else>
          <a-button @click="cancelEditing"> 取消 </a-button>
          <a-button type="primary" @click="saveChanges"> 保存 </a-button>
        </template>
      </div>
    </a-card>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { updateUserUsingPost } from '@/api/userController.ts'
import { message } from 'ant-design-vue'

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
  userProfile: loginUserStore.loginUser?.userProfile || '',
})

// 编辑状态
const isEditing = ref(false)

// 原始用户信息备份，用于取消操作
const originalUserInfo = { ...userInfo }

// 开始编辑
const startEditing = () => {
  // 保存原始数据
  Object.assign(originalUserInfo, userInfo)
  isEditing.value = true
}

// 取消编辑
const cancelEditing = () => {
  // 恢复原始数据
  Object.assign(userInfo, originalUserInfo)
  isEditing.value = false
}

// 保存修改
const saveChanges = async () => {
  try {
    // 调用更新用户信息接口
    const res = await updateUserUsingPost({
      userAccount: userInfo.userAccount,
      userName: userInfo.userName,
      userProfile: userInfo.userProfile,
    })
    // 检查响应
    if (res.data.code === 0) {
      // 更新成功后退出编辑状态
      isEditing.value = false
      message.success('保存成功')
    } else {
      message.error('保存失败: ' + (res.data.message || '未知错误'))
    }
  } catch (error: any) {
    console.error('更新用户信息失败:', error)
    message.error('保存失败: ' + (error.message || '网络错误'))
  }
}
</script>

<style scoped>
/* 蓝白色调样式 */
#personalInfoPage {
  max-width: 600px;
  margin: 0 auto;
  padding: 40px 24px;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.title {
  text-align: center;
  margin-bottom: 32px;
  color: #1890ff;
  font-weight: 600;
  font-size: 24px;
}

/* 卡片样式优化，蓝白色调 */
.info-card {
  width: 100%;
  max-width: 480px;
  padding: 40px;
  border-radius: 12px;
  background-color: #ffffff;
  box-shadow: 0 4px 16px rgba(24, 144, 255, 0.1);
  border: 1px solid #e6f7ff;
}

/* 头像容器样式 */
.avatar-container {
  display: flex;
  justify-content: center;
  margin-bottom: 32px;
}

/* 表单样式优化 */
.a-form {
  width: 100%;
}

.a-form-item {
  margin-bottom: 20px;
}

.a-form-item-label {
  font-weight: 500;
  color: #333;
}

.a-input,
.a-textarea {
  border-radius: 6px;
  border: 1px solid #d9d9d9;
  transition: all 0.3s;
}

.a-input:focus,
.a-textarea:focus {
  border-color: #1890ff;
  box-shadow: 0 0 0 2px rgba(24, 144, 255, 0.2);
}

/* 按钮样式 */
.button-container {
  display: flex;
  justify-content: flex-end;
  margin-top: 32px;
  gap: 16px;
}

.a-button {
  border-radius: 6px;
  padding: 8px 20px;
  font-weight: 500;
}

.a-button-primary {
  background-color: #1890ff;
  border-color: #1890ff;
}

.a-button-primary:hover {
  background-color: #40a9ff;
  border-color: #40a9ff;
}

/* 编辑状态下的表单样式 */
.edit-mode .a-form-item {
  margin-bottom: 20px;
}
</style>
