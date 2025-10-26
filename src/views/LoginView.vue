<template>
  <div class="login-container">
    <div class="login-form">
      <h1 class="login-title">学生管理系统</h1>
      <div class="form-group">
        <label for="username">用户名</label>
        <input 
          type="text" 
          id="username" 
          v-model="username" 
          placeholder="请输入用户名"
          class="form-input"
        />
      </div>
      <div class="form-group">
        <label for="password">密码</label>
        <input 
          type="password" 
          id="password" 
          v-model="password" 
          placeholder="请输入密码"
          class="form-input"
        />
      </div>
      <button @click="handleLogin" class="login-button">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="m9 18 6-6-6-6"/>
        </svg>
        登录
      </button>
      <p v-if="error" class="error-message">{{ error }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

// 路由实例
const router = useRouter()

// 响应式数据
const username = ref('admin')
const password = ref('123456')
const error = ref('')

// 登录处理函数
const handleLogin = () => {
  if (username.value === 'admin' && password.value === '123456') {
    localStorage.setItem('isAuthenticated', 'true')
    router.push({ name: 'students' })
  } else {
    error.value = '用户名或密码错误'
  }
}
</script>

<style scoped>
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  background-size: cover;
  background-position: center;
  position: relative;
  overflow: hidden;
  width: 100%;
}

/* 为大屏添加装饰元素 */
.login-container::before,
.login-container::after {
  content: '';
  position: absolute;
  width: 300px;
  height: 300px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  z-index: 0;
}

/* 调整装饰元素在不同屏幕尺寸的大小 */
@media (max-width: 768px) {
  .login-container::before,
  .login-container::after {
    width: 200px;
    height: 200px;
  }
}

.login-container::before {
  top: 10%;
  left: 10%;
  transform: scale(1.2);
  animation: float 8s ease-in-out infinite;
}

.login-container::after {
  bottom: 10%;
  right: 10%;
  transform: scale(1.5);
  animation: float 10s ease-in-out infinite reverse;
}

@keyframes float {
  0%, 100% {
    transform: translateY(0) scale(1);
  }
  50% {
    transform: translateY(-20px) scale(1.05);
  }
}

.login-form {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 40px;
  width: 100%;
  max-width: 400px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  position: relative;
  z-index: 1;
  /* 确保表单不会在大屏上变得太小 */
  border: 1px solid rgba(255, 255, 255, 0.2);
  margin: 0 auto;
}

/* 大屏优化 - 修复1024px以上的布局问题 */
@media (min-width: 768px) and (max-width: 1199px) {
  .login-form {
    max-width: 420px;
    padding: 45px;
  }
}

@media (min-width: 1200px) {
  .login-form {
    max-width: 450px;
    padding: 50px;
    box-shadow: 0 25px 50px rgba(0, 0, 0, 0.15);
  }
}

.login-form:hover {
  transform: translateY(-5px);
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.15);
}

.login-title {
  text-align: center;
  color: #333;
  font-size: 28px;
  margin-bottom: 30px;
  font-weight: 700;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  color: #555;
  font-weight: 500;
}

.form-input {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  font-size: 16px;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
  background-color: rgba(255, 255, 255, 0.8);
}

.form-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.login-button {
  width: 100%;
  padding: 12px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.3s ease, transform 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.login-button:hover {
  background: linear-gradient(135deg, #5a67d8 0%, #6b46c1 100%);
  transform: translateY(-2px);
}

.login-button:active {
  transform: translateY(0);
}

.error-message {
  color: #e53e3e;
  text-align: center;
  margin-top: 15px;
  font-size: 14px;
}

@media (max-width: 480px) {
  .login-form {
    padding: 30px 20px;
  }
  
  .login-title {
    font-size: 24px;
  }
}
</style>