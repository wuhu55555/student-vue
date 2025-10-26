<template>
  <div class="student-management-container">
    <header class="header">
      <h1 class="header-title">学生管理系统</h1>
      <button @click="handleLogout" class="logout-button">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/>
          <polyline points="16 17 21 12 16 7"/>
          <line x1="21" y1="12" x2="9" y2="12"/>
        </svg>
        退出登录
      </button>
    </header>

    <main class="main-content">
      <div class="stats-section">
        <div class="stat-card">
          <h3>学生总数</h3>
          <p class="stat-value">{{ students.length }}</p>
        </div>
      </div>

      <div class="add-student-section">
        <h2>添加学生</h2>
        <div class="add-form">
          <input 
            type="text" 
            v-model="newStudent.name" 
            placeholder="姓名"
            class="form-input"
          />
          <input 
            type="number" 
            v-model="newStudent.age" 
            placeholder="年龄"
            class="form-input"
            min="1"
          />
          <input 
            type="text" 
            v-model="newStudent.class" 
            placeholder="班级"
            class="form-input"
          />
          <button @click="addStudent" class="add-button">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="12" y1="5" x2="12" y2="19"/>
              <line x1="5" y1="12" x2="19" y2="12"/>
            </svg>
            添加
          </button>
        </div>
      </div>

      <div class="students-section">
        <h2>学生列表</h2>
        <div class="students-grid">
          <div v-if="students.length === 0" class="empty-state">
            <p>暂无学生数据</p>
          </div>
          <div v-for="student in students" :key="student._id" class="student-card">
            <div class="student-info">
              <h3 class="student-name">{{ student.name }}</h3>
              <p class="student-detail">年龄: {{ student.age }}</p>
              <p class="student-detail">班级: {{ student.class }}</p>
            </div>
            <div class="student-actions">
              <button @click="editStudent(student)" class="edit-button">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/>
                  <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/>
                </svg>
              </button>
              <button @click="deleteStudent(student.id)" class="delete-button">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <polyline points="3 6 5 6 21 6"/>
                  <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/>
                  <line x1="10" y1="11" x2="10" y2="17"/>
                  <line x1="14" y1="11" x2="14" y2="17"/>
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- 编辑学生模态框 -->
    <div v-if="showEditModal" class="modal-overlay" @click.self="closeEditModal">
      <div class="modal-content">
        <h2>编辑学生信息</h2>
        <div class="modal-form">
          <input 
            type="text" 
            v-model="editingStudent.name" 
            placeholder="姓名"
            class="form-input"
          />
          <input 
            type="number" 
            v-model="editingStudent.age" 
            placeholder="年龄"
            class="form-input"
            min="1"
          />
          <input 
            type="text" 
            v-model="editingStudent.class" 
            placeholder="班级"
            class="form-input"
          />
        </div>
        <div class="modal-actions">
          <button @click="closeEditModal" class="cancel-button">取消</button>
          <button @click="updateStudent" class="save-button">保存</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'

// 路由实例
const router = useRouter()

// API基地址
const apiBaseUrl = 'https://igskwbawrefi.sealoshzh.site/api'

// 状态定义
const students = ref([])
const newStudent = reactive({
  name: '',
  age: '',
  class: ''
})
const editingStudent = reactive({
  id: '',
  name: '',
  age: '',
  class: ''
})
const showEditModal = ref(false)

// 获取学生列表
const fetchStudents = async () => {
  try {
    const response = await axios.get(`${apiBaseUrl}/get-students`)
    // 将_id映射为id以便在模板中使用
    students.value = response.data.map(student => ({
      ...student,
      id: student._id
    }))
  } catch (error) {
    console.error('获取学生列表失败:', error)
    alert('获取学生列表失败，请稍后重试')
  }
}

// 添加学生
const addStudent = async () => {
  if (!newStudent.name || !newStudent.age || !newStudent.class) {
    alert('请填写所有字段')
    return
  }
  
  // 验证年龄是否为正整数
  const age = Number(newStudent.age)
  if (!Number.isInteger(age) || age <= 0) {
    alert('年龄必须是正整数')
    return
  }
  
  try {
    const studentData = {
      name: newStudent.name,
      age: age,
      class: newStudent.class
    }
    
    const response = await axios.post(`${apiBaseUrl}/add-student`, studentData)
    
    // 添加到列表中，并映射_id为id
    const student = {
      ...response.data,
      id: response.data._id
    }
    students.value.push(student)
    
    // 重置表单
    newStudent.name = ''
    newStudent.age = ''
    newStudent.class = ''
    
    alert('学生添加成功')
  } catch (error) {
    console.error('添加学生失败:', error)
    const errorMsg = error.response?.data?.message || '添加学生失败，请稍后重试'
    const errorDetails = error.response?.data?.errors
    if (errorDetails && errorDetails.length > 0) {
      alert(errorMsg + ':\n' + errorDetails.join('\n'))
    } else {
      alert(errorMsg)
    }
  }
}

// 编辑学生
const editStudent = (student) => {
  editingStudent.id = student.id
  editingStudent.name = student.name
  editingStudent.age = student.age
  editingStudent.class = student.class
  showEditModal.value = true
}

// 更新学生
const updateStudent = async () => {
  if (!editingStudent.name || !editingStudent.age || !editingStudent.class) {
    alert('请填写所有字段')
    return
  }
  
  // 验证年龄是否为正整数
  const age = Number(editingStudent.age)
  if (!Number.isInteger(age) || age <= 0) {
    alert('年龄必须是正整数')
    return
  }
  
  try {
    const updateData = {
      name: editingStudent.name,
      age: age,
      class: editingStudent.class
    }
    
    const response = await axios.post(`${apiBaseUrl}/update-student/${editingStudent.id}`, updateData)
    
    // 更新本地列表
    const index = students.value.findIndex(s => s.id === editingStudent.id)
    if (index !== -1) {
      students.value[index] = {
        ...response.data,
        id: response.data._id
      }
    }
    
    alert('学生信息更新成功')
    closeEditModal()
  } catch (error) {
    console.error('更新学生失败:', error)
    const errorMsg = error.response?.data?.message || '更新学生失败，请稍后重试'
    const errorDetails = error.response?.data?.errors
    if (errorDetails && errorDetails.length > 0) {
      alert(errorMsg + ':\n' + errorDetails.join('\n'))
    } else {
      alert(errorMsg)
    }
  }
}

// 删除学生
const deleteStudent = async (id) => {
  if (confirm('确定要删除这个学生吗？')) {
    try {
      await axios.post(`${apiBaseUrl}/del-student/${id}`)
      
      // 从本地列表中删除
      students.value = students.value.filter(student => student.id !== id)
      
      alert('学生删除成功')
    } catch (error) {
      console.error('删除学生失败:', error)
      const errorMsg = error.response?.data?.message || '删除学生失败，请稍后重试'
      alert(errorMsg)
    }
  }
}

// 关闭编辑模态框
const closeEditModal = () => {
  showEditModal.value = false
  editingStudent.id = ''
  editingStudent.name = ''
  editingStudent.age = ''
  editingStudent.class = ''
}

// 退出登录
const handleLogout = () => {
  localStorage.removeItem('isAuthenticated')
  router.push({ name: 'login' })
}

// 组件挂载时获取学生列表
onMounted(() => {
  fetchStudents()
})
</script>

<style scoped>
.student-management-container {
  min-height: 100vh;
  background-color: #f5f7fa;
  display: flex;
  flex-direction: column;
  width: 100%;
}

.header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px 40px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 100%;
  box-sizing: border-box;
}

.header-title {
  font-size: 24px;
  font-weight: 700;
  margin: 0;
}

.logout-button {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: background 0.3s ease, transform 0.3s ease;
  backdrop-filter: blur(10px);
}

.logout-button:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
}

.main-content {
  padding: 40px;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
  box-sizing: border-box;
  flex: 1;
}

/* 确保在各种屏幕尺寸下都正确居中 */
@media (max-width: 768px) {
  .main-content {
    padding: 20px;
  }
}

/* 大屏优化 */
@media (min-width: 1400px) {
  .main-content {
    max-width: 1400px;
    padding: 50px;
  }
  
  .students-grid {
    grid-template-columns: repeat(auto-fill, minmax(400px, 1fr));
  }
  
  .add-student-section, .students-section {
    padding: 40px;
  }
}

.stats-section {
  display: flex;
  gap: 20px;
  margin-bottom: 30px;
}

.stat-card {
  background: white;
  border-radius: 16px;
  padding: 24px;
  flex: 1;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.stat-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.12);
}

.stat-card h3 {
  color: #667eea;
  font-size: 16px;
  margin: 0 0 10px 0;
  font-weight: 500;
}

.stat-value {
  font-size: 36px;
  font-weight: 700;
  color: #333;
  margin: 0;
}

.add-student-section, .students-section {
  background: white;
  border-radius: 16px;
  padding: 30px;
  margin-bottom: 30px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
}

h2 {
  color: #333;
  margin: 0 0 20px 0;
  font-size: 22px;
  font-weight: 600;
}

.add-form {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
  justify-content: center;
  max-width: 100%;
}

.form-input {
  flex: 1;
  min-width: 200px;
  max-width: 100%;
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  font-size: 16px;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

.form-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.add-button {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 10px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: background 0.3s ease, transform 0.3s ease;
  white-space: nowrap;
}

.add-button:hover {
  background: linear-gradient(135deg, #5a67d8 0%, #6b46c1 100%);
  transform: translateY(-2px);
}

.students-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  gap: 20px;
}

.empty-state {
  grid-column: 1 / -1;
  text-align: center;
  padding: 60px 20px;
  color: #999;
  font-size: 18px;
}

.student-card {
  background: #f8f9fa;
  border-radius: 12px;
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  border: 1px solid #e9ecef;
}

.student-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
}

.student-name {
  color: #333;
  font-size: 18px;
  font-weight: 600;
  margin: 0 0 8px 0;
}

.student-detail {
  color: #666;
  margin: 4px 0;
  font-size: 14px;
}

.student-actions {
  display: flex;
  gap: 10px;
}

.edit-button, .delete-button {
  padding: 8px 12px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s ease, transform 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.edit-button {
  background: #3182ce;
  color: white;
}

.edit-button:hover {
  background: #2c5aa0;
  transform: translateY(-1px);
}

.delete-button {
  background: #e53e3e;
  color: white;
}

.delete-button:hover {
  background: #c53030;
  transform: translateY(-1px);
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  border-radius: 16px;
  padding: 30px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
}

.modal-form {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 20px;
}

.modal-actions {
  display: flex;
  gap: 15px;
  justify-content: flex-end;
}

.cancel-button, .save-button {
  padding: 10px 20px;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  font-size: 16px;
  font-weight: 600;
  transition: background 0.3s ease, transform 0.3s ease;
}

.cancel-button {
  background: #e2e8f0;
  color: #4a5568;
}

.cancel-button:hover {
  background: #cbd5e0;
}

.save-button {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.save-button:hover {
  background: linear-gradient(135deg, #5a67d8 0%, #6b46c1 100%);
  transform: translateY(-1px);
}

@media (max-width: 768px) {
  .header {
    padding: 15px 20px;
  }
  
  .header-title {
    font-size: 20px;
  }
  
  .main-content {
    padding: 20px;
  }
  
  .add-form {
    flex-direction: column;
    align-items: center;
  }
  
  .form-input {
    min-width: unset;
    width: 100%;
    max-width: 400px;
  }
  
  .add-button {
    width: 100%;
    max-width: 400px;
  }
  
  .students-grid {
    grid-template-columns: 1fr;
  }
  
  .student-card {
    flex-direction: column;
    align-items: flex-start;
    gap: 15px;
  }
  
  .student-actions {
    align-self: flex-end;
  }
}
</style>