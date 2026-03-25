<template>
  <div id="userManagePage">
    <!-- 搜索表单 -->
    <a-form
      layout="inline"
      :model="searchParams"
      @finish="doSearch"
      style="width: 100%; display: flex; justify-content: space-between; align-items: center"
    >
      <div style="display: flex; align-items: center">
        <a-form-item label="账号">
          <a-input v-model:value="searchParams.userAccount" placeholder="输入账号" allow-clear />
        </a-form-item>
        <a-form-item label="用户名">
          <a-input v-model:value="searchParams.userName" placeholder="输入用户名" allow-clear />
        </a-form-item>
        <a-form-item>
          <a-button type="primary" html-type="submit">搜索</a-button>
        </a-form-item>
      </div>
      <a-form-item>
        <a-button type="primary" @click="showAddModal">新增用户</a-button>
      </a-form-item>
    </a-form>
    <div style="margin-bottom: 16px" />
    <!-- 表格 -->
    <a-table
      :columns="columns"
      :data-source="dataList"
      :pagination="pagination"
      @change="doTableChange"
      style="min-height: 400px"
    >
      <template #bodyCell="{ column, record }">
        <template v-if="column.dataIndex === 'userAvatar'">
          <a-avatar :style="{ backgroundColor: '#1890ff' }" size="large">
            {{ (record.userName || '无').charAt(0) }}
          </a-avatar>
        </template>
        <template v-else-if="column.dataIndex === 'userRole'">
          <template v-if="editingRow === record.id">
            <a-select v-model:value="editForm[record.id].userRole" style="width: 120px">
              <a-select-option value="admin">管理员</a-select-option>
              <a-select-option value="user">普通用户</a-select-option>
            </a-select>
          </template>
          <template v-else>
            <div v-if="record.userRole === 'admin'">
              <a-tag color="green">管理员</a-tag>
            </div>
            <div v-else>
              <a-tag color="blue">普通用户</a-tag>
            </div>
          </template>
        </template>
        <template
          v-else-if="
            column.dataIndex === 'userAccount' ||
            column.dataIndex === 'userName' ||
            column.dataIndex === 'userProfile'
          "
        >
          <template v-if="editingRow === record.id">
            <a-input v-model:value="editForm[record.id][column.dataIndex]" />
          </template>
          <template v-else>
            {{ record[column.dataIndex] }}
          </template>
        </template>
        <template v-if="column.dataIndex === 'createTime'">
          {{ dayjs(record.createTime).format('YYYY-MM-DD HH:mm:ss') }}
        </template>
        <template v-else-if="column.key === 'action'">
          <template v-if="editingRow === record.id">
            <a-button type="primary" style="margin-right: 8px" @click="doSave(record.id)"
              >保存</a-button
            >
            <a-button style="margin-right: 8px" @click="doCancel(record.id)">取消</a-button>
          </template>
          <template v-else>
            <a-button type="primary" style="margin-right: 8px" @click="doEdit(record)"
              >编辑</a-button
            >
            <a-button danger @click="doDelete(record.id)">删除</a-button>
          </template>
        </template>
      </template>
    </a-table>

    <!-- 新增用户弹窗 -->
    <a-modal
      v-model:visible="addModalVisible"
      title="新增用户"
      @ok="handleAddOk"
      @cancel="handleAddCancel"
    >
      <a-form ref="addFormRef" :model="addForm" layout="vertical" name="addUserForm">
        <a-form-item
          name="userAccount"
          label="用户账号"
          :rules="[{ required: true, message: '请输入用户账号' }]"
        >
          <a-input v-model:value="addForm.userAccount" placeholder="请输入用户账号" />
        </a-form-item>
        <a-form-item
          name="userPassword"
          label="用户密码"
          :rules="[
            { required: true, message: '请输入用户密码' },
            { min: 8, message: '密码不能小于 8 位' },
          ]"
        >
          <a-input-password v-model:value="addForm.userPassword" placeholder="请输入用户密码" />
        </a-form-item>
        <a-form-item
          name="checkPassword"
          label="密码确认"
          :rules="[
            { required: true, message: '请输入确认密码' },
            { min: 8, message: '确认密码不能小于 8 位' },
          ]"
        >
          <a-input-password v-model:value="addForm.checkPassword" placeholder="请确认密码" />
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>
<script lang="ts" setup>
import { computed, onMounted, reactive, ref } from 'vue'
import {
  addUserUsingPost,
  deleteUserUsingPost,
  listUserVoByPageUsingPost,
  updateUserUsingPost,
} from '@/api/userController.ts'
import { message, Modal } from 'ant-design-vue'
import dayjs from 'dayjs'

const columns = [
  {
    title: 'id',
    dataIndex: 'id',
  },
  {
    title: '账号',
    dataIndex: 'userAccount',
  },
  {
    title: '用户名',
    dataIndex: 'userName',
  },
  {
    title: '头像',
    dataIndex: 'userAvatar',
  },
  {
    title: '简介',
    dataIndex: 'userProfile',
  },
  {
    title: '用户角色',
    dataIndex: 'userRole',
  },
  {
    title: '创建时间',
    dataIndex: 'createTime',
  },
  {
    title: '操作',
    key: 'action',
  },
]

// 定义数据
const dataList = ref<API.UserVO[]>([])
const total = ref(0)

// 搜索条件
const searchParams = reactive<API.UserQueryQo>({
  current: 1,
  pageSize: 10,
  sortField: 'createTime',
  sortOrder: 'ascend',
})

// 获取数据
const fetchData = async () => {
  const res = await listUserVoByPageUsingPost({
    ...searchParams,
  })
  if (res.data.code === 0 && res.data.data) {
    dataList.value = res.data.data.records ?? []
    total.value = res.data.data.total ?? 0
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
}

// 页面加载时获取数据，请求一次
onMounted(() => {
  fetchData()
})

// 分页参数
const pagination = computed(() => {
  return {
    current: searchParams.current,
    pageSize: searchParams.pageSize,
    total: total.value,
    showSizeChanger: true,
    showTotal: (total) => `共 ${total} 条`,
  }
})

// 表格变化之后，重新获取数据
const doTableChange = (page: any) => {
  searchParams.current = page.current
  searchParams.pageSize = page.pageSize
  fetchData()
}

// 搜索数据
const doSearch = () => {
  // 重置页码
  searchParams.current = 1
  fetchData()
}

// 编辑状态管理
const editingRow = ref<string | null>(null)
const editForm = reactive<Record<string, any>>({})
const originalData = ref<Record<string, any>>({})

// 新增用户状态管理
const addModalVisible = ref(false)
const addFormRef = ref()
const addForm = reactive({
  userAccount: '',
  userPassword: '',
  checkPassword: '',
})

// 显示新增用户弹窗
const showAddModal = () => {
  // 重置表单数据
  addForm.userAccount = ''
  addForm.userPassword = ''
  addForm.checkPassword = ''
  // 显示弹窗
  addModalVisible.value = true
}

// 开始编辑
const doEdit = (record: API.UserVO) => {
  editingRow.value = record.id
  // 保存原始数据
  originalData.value[record.id] = { ...record }
  // 初始化编辑表单数据
  editForm[record.id] = { ...record }
}

// 取消编辑
const doCancel = (id: string) => {
  editingRow.value = null
  // 清除编辑数据
  delete editForm[id]
  delete originalData.value[id]
}

// 保存编辑
const doSave = async (id: string) => {
  const formData = editForm[id]
  if (!formData) {
    return
  }

  // 表单验证
  if (!formData.userAccount) {
    message.error('账号不能为空')
    return
  }
  if (!formData.userName) {
    message.error('用户名不能为空')
    return
  }
  if (!formData.userRole) {
    message.error('用户角色不能为空')
    return
  }

  try {
    const res = await updateUserUsingPost(formData)
    if (res.data.code === 0) {
      message.success('保存成功')
      // 恢复到非编辑模式
      editingRow.value = null
      // 清除编辑数据
      delete editForm[id]
      delete originalData.value[id]
      // 刷新数据
      fetchData()
    } else {
      message.error('保存失败：' + res.data.message)
    }
  } catch (error) {
    message.error('保存失败：网络错误')
  }
}

// 删除数据
const doDelete = (id: string) => {
  if (!id) {
    return
  }
  Modal.confirm({
    title: '确认删除',
    content: '确定要删除该用户吗？此操作不可恢复。',
    okText: '确认',
    cancelText: '取消',
    onOk: async () => {
      const res = await deleteUserUsingPost({ id })
      if (res.data.code === 0) {
        message.success('删除成功')
        // 刷新数据
        fetchData()
      } else {
        message.error('删除失败')
      }
    },
  })
}

// 新增用户保存
const handleAddOk = async () => {
  // 表单验证
  if (addFormRef.value) {
    await addFormRef.value
      .validate()
      .then(async () => {
        // 检查两次密码是否一致
        if (addForm.userPassword !== addForm.checkPassword) {
          message.error('两次输入的密码不一致')
          return
        }

        try {
          const res = await addUserUsingPost({
            userAccount: addForm.userAccount,
            userPassword: addForm.userPassword,
            checkPassword: addForm.checkPassword,
          })
          if (res.data.code === 0) {
            message.success('新增成功')
            // 关闭弹窗
            addModalVisible.value = false
            // 刷新数据
            fetchData()
          } else {
            message.error('新增失败：' + res.data.message)
          }
        } catch (error) {
          message.error('新增失败：网络错误')
        }
      })
      .catch((error) => {
        console.log('表单验证失败:', error)
      })
  }
}

// 新增用户取消
const handleAddCancel = () => {
  // 关闭弹窗
  addModalVisible.value = false
  // 重置表单数据
  addForm.userAccount = ''
  addForm.userPassword = ''
  addForm.checkPassword = ''
}
</script>
