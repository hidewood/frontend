<template>
  <div class="auth-container">
    <!-- 登录表单 -->
    <el-card v-if="isLoginForm" class="form-card">
      <template #header>
        <h2 class="form-title">用户登录</h2>
      </template>
      
      <el-form 
        :model="loginForm" 
        :rules="loginRules" 
        ref="loginFormRef"
        @keyup.enter="handleLogin"
      >
      <el-form-item prop="username">
      <el-input
        v-model="loginForm.username"
        placeholder="请输入用户名"
        prefix-icon="User"  
      />
      </el-form-item>

        <el-form-item prop="password">
          <el-input
            v-model="loginForm.password"
            type="password"
            placeholder="请输入密码"
            show-password
            prefix-icon="Lock"
          />
        </el-form-item>

        <el-form-item>
          <el-button 
            type="primary" 
            class="submit-btn"
            @click="handleLogin"
          >
            立即登录
          </el-button>
          <div class="form-switch">
            没有账号？
            <el-link type="primary" @click="toggleForm">立即注册</el-link>
          </div>
        </el-form-item>
      </el-form>
    </el-card>

    <!-- 注册表单 -->
    <el-card v-else class="form-card">
      <template #header>
        <h2 class="form-title">用户注册</h2>
      </template>

      <el-form
        :model="registerForm"
        :rules="registerRules"
        ref="registerFormRef"
      >
        <el-form-item prop="username">
          <el-input
            v-model="registerForm.username"
            placeholder="请输入用户名（4-16位字符）"
            prefix-icon="User"
          />
        </el-form-item>

        <el-form-item prop="email">
          <el-input
            v-model="registerForm.email"
            placeholder="请输入邮箱"
            prefix-icon="Message"
          />
        </el-form-item>

        <el-form-item prop="password">
          <el-input
            v-model="registerForm.password"
            type="password"
            placeholder="请输入密码（6-20位）"
            show-password
            prefix-icon="Lock"
          />
        </el-form-item>

        <el-form-item prop="confirmPassword">
          <el-input
            v-model="registerForm.confirmPassword"
            type="password"
            placeholder="请确认密码"
            show-password
            prefix-icon="Lock"
          />
        </el-form-item>

        <el-form-item>
          <el-button 
            type="success" 
            class="submit-btn"
            @click="handleRegister"
          >
            立即注册
          </el-button>
          <div class="form-switch">
            已有账号？
            <el-link type="primary" @click="toggleForm">立即登录</el-link>
          </div>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { ElMessage } from 'element-plus'
import axios from 'axios'

// 表单切换状态
const isLoginForm = ref(true)

// 登录表单相关
const loginFormRef = ref()
const loginForm = reactive({
  username: '',
  password: ''
})

const loginRules = {
  username: [  // 字段名修正
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 4, max: 16, message: '用户名长度4-16位', trigger: 'blur' }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, max: 20, message: '长度在6到20个字符', trigger: 'blur' }
  ]
}

// 注册表单相关
const registerFormRef = ref()
const registerForm = reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
})

const validateUsername = (rule, value, callback) => {
  const reg = /^[a-zA-Z0-9_-]{4,16}$/
  if (!reg.test(value)) {
    callback(new Error('用户名应为4-16位字母、数字或下划线'))
  } else {
    callback()
  }
}

const validateConfirmPassword = (rule, value, callback) => {
  if (value !== registerForm.password) {
    callback(new Error('两次输入密码不一致'))
  } else {
    callback()
  }
}

const registerRules = {
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { validator: validateUsername, trigger: 'blur' }
  ],
  email: [
    { required: true, message: '请输入邮箱地址', trigger: 'blur' },
    { type: 'email', message: '请输入正确的邮箱格式', trigger: ['blur', 'change'] }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, max: 20, message: '长度在6到20个字符', trigger: 'blur' }
  ],
  confirmPassword: [
    { required: true, message: '请再次输入密码', trigger: 'blur' },
    { validator: validateConfirmPassword, trigger: 'blur' }
  ]
}

// 切换表单
const toggleForm = () => {
  isLoginForm.value = !isLoginForm.value
  // 清空表单验证
  loginFormRef.value?.resetFields()
  registerFormRef.value?.resetFields()
}

// 处理登录
const handleLogin = async () => {
  try {
    await loginFormRef.value.validate()
    
    const response = await axios.post('http://localhost:5000/api/auth/login', {
      username: loginForm.username,
      password: loginForm.password
    })

    if (response.data.code === 200) {
      ElMessage.success('登录成功')
      // 处理登录成功逻辑，例如保存token
      localStorage.setItem('token', response.data.token)
      // 跳转到首页
      window.location.href = '/'
    } else {
      ElMessage.error(response.data.message)
    }
  } catch (error) {
    if (error.response) {
      ElMessage.error(error.response.data.message || '登录失败')
    } else if (error.message !== 'VALIDATE_ERROR') {
      ElMessage.error('网络请求异常')
    }
  }
}

// 处理注册
const handleRegister = async () => {
  try {
    await registerFormRef.value.validate()
    
    const response = await axios.post('http://localhost:5000/api/auth/register', {
      username: registerForm.username,
      email: registerForm.email,
      password: registerForm.password
    })

    if (response.data.code === 200) {
      ElMessage.success('注册成功')
      toggleForm() // 切换到登录表单
    } else {
      ElMessage.error(response.data.message)
    }
  } catch (error) {
    if (error.response) {
      ElMessage.error(error.response.data.message || '注册失败')
    } else if (error.message !== 'VALIDATE_ERROR') {
      ElMessage.error('网络请求异常')
    }
  }
}
</script>

<style scoped>
.auth-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: #f0f2f5;
}

.form-card {
  width: 400px;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.1);
}

.form-title {
  text-align: center;
  color: #303133;
}

.submit-btn {
  width: 100%;
  margin-top: 10px;
}

.form-switch {
  margin-top: 15px;
  text-align: center;
  font-size: 14px;
}

.el-form-item {
  margin-bottom: 22px;
}
</style>