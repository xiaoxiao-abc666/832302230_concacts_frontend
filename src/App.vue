<script setup>
// 1. 导入工具
import { ref, onMounted, computed } from 'vue' // 【新】导入 computed
import axios from 'axios'

// 2. "魔法盒子"
const message = ref('正在从后端加载消息...')
const contactsList = ref([]) // 存放联系人列表

// --- 表单相关的 "魔法盒子" ---
// 'new' 前缀改成 'form'，因为这个表单现在既用于"新建"也用于"修改"
const formName = ref('')
const formPhone = ref('')

// 【新功能】一个 "魔法盒子" 用来追踪我们正在修改的联系人 ID
// null 表示“不在修改模式”
// 某个数字 (id) 表示“正在修改这个 ID 的联系人”
const editingContactId = ref(null)

// 3. 【新功能】一个“计算属性”，用来判断当前是否处于“修改模式”
//    它会自动根据 editingContactId 的值返回 true 或 false
const isEditing = computed(() => {
  return editingContactId.value !== null
})

// 4. "获取所有联系人" 函数 (不变)
async function fetchContacts() {
  try {
    const response = await axios.get('http://127.0.0.1:5000/api/contacts')
    contactsList.value = response.data
    console.log('联系人列表已更新:', contactsList.value)
  } catch (error) {
    console.error('获取联系人列表失败!', error)
    alert('获取联系人列表失败!')
  }
}

// 5. 页面加载时 (不变)
onMounted(() => {
  axios.get('http://127.0.0.1:5000/api/test')
    .then(response => { message.value = response.data.message })
    .catch(error => { message.value = '连接后端失败 :(' })
  
  fetchContacts() // 获取列表
})

// 6. 【重大修改】"提交表单" 函数
//    这个函数现在变得更智能了
async function handleSubmitForm() {
  // (A) 验证表单
  if (!formName.value || !formPhone.value) {
    alert('姓名和电话都不能为空！')
    return
  }

  // (B) 准备数据
  const contactData = {
    name: formName.value,
    phone: formPhone.value
  }

  try {
    // (C) 【关键逻辑】判断是“修改”还是“新建”
    if (isEditing.value) {
      // --- (1) 是“修改”模式 ---
      const id = editingContactId.value
      await axios.put(`http://127.0.0.1:5000/api/contacts/${id}`, contactData)
      alert('修改成功！')
    } else {
      // --- (2) 是“新建”模式 ---
      await axios.post('http://127.0.0.1:5000/api/contacts', contactData)
      alert('添加成功！')
    }
    
    // (D) 无论“修改”还是“新建”，成功后都要：
    clearForm()     // 清空表单
    fetchContacts() // 重新获取列表

  } catch (error) {
    console.error('操作失败!', error)
    alert('操作失败，请查看控制台信息。')
  }
}

// 7. 【新功能】"清空表单" 和 "退出修改模式" 的辅助函数
function clearForm() {
  formName.value = ''
  formPhone.value = ''
  editingContactId.value = null // 退出“修改模式”
}

// 8. 【新功能】"点击修改按钮" 时触发的函数
function handleStartEdit(contact) {
  // (A) 进入“修改模式”，并记录 ID
  editingContactId.value = contact.id
  
  // (B) 把这个联系人的旧数据，填充到表单里
  formName.value = contact.name
  formPhone.value = contact.phone
}

// 9. "删除联系人" 函数 (不变)
async function handleDeleteContact(contactId) {
  if (!confirm('你确定要删除这个联系人吗？')) {
    return
  }
  try {
    await axios.delete(`http://127.0.0.1:5000/api/contacts/${contactId}`)
    alert('删除成功！')
    fetchContacts() // 重新获取列表
  } catch (error) {
    console.error('删除联系人出错了!', error)
    alert('删除失败，请查看控制台信息。')
  }
}

</script>

<template>
  <div id="app">
    <h1>{{ message }}</h1>

    <form @submit.prevent="handleSubmitForm" class="contact-form">
      
      <h2>{{ isEditing ? '正在修改联系人' : '添加新联系人' }}</h2>
      
      <div class="form-group">
        <label for="name">姓名:</label>
        <input type="text" id="name" v-model="formName" placeholder="输入姓名">
      </div>
      <div class="form-group">
        <label for="phone">电话:</label>
        <input type="text" id="phone" v-model="formPhone" placeholder="输入电话号码">
      </div>
      
      <button type="submit">{{ isEditing ? '确认修改' : '点击添加' }}</button>
      
      <button v-if="isEditing" @click.prevent="clearForm" class="cancel-button">
        取消
      </button>
    </form>

    <hr> <div class="contacts-list">
      <h2>我的通讯录</h2>
      <div v-if="contactsList.length === 0" class="empty-list">
        暂无联系人
      </div>
      
      <ul v-else>
        <li v-for="contact in contactsList" :key="contact.id" class="contact-item">
          <div class="contact-info">
            <strong>{{ contact.name }}</strong> - <span>{{ contact.phone }}</span>
          </div>
          
          <div class="contact-buttons">
            <button @click="handleStartEdit(contact)" class="edit-button">
              修改
            </button>
            
            <button @click="handleDeleteContact(contact.id)" class="delete-button">
              删除
            </button>
          </div>
        </li>
      </ul>
    </div>

  </div>
</template>

<style>
/* ... 你之前的样式代码 ... */
#app {
  width: 400px;
  margin: 40px auto;
  text-align: center;
  font-family: Arial, sans-serif;
}
.contact-form {
  border: 1px solid #ccc;
  padding: 20px;
  border-radius: 8px;
  background-color: #f9f9f9;
  margin-bottom: 20px;
}
.form-group {
  margin-bottom: 15px;
  text-align: left;
}
.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}
.form-group input {
  width: 95%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}
button {
  background-color: #007bff;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin: 0 5px; /* 增加一点按钮间距 */
}
button:hover {
  background-color: #0056b3;
}
.contacts-list {
  border: 1px solid #eee;
  padding: 20px;
  border-radius: 8px;
}
.empty-list {
  color: #888;
}
ul {
  list-style: none;
  padding: 0;
}
.contact-item {
  background-color: #fefefe;
  border: 1px solid #f0f0f0;
  padding: 10px;
  margin-bottom: 8px;
  border-radius: 4px;
  text-align: left;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.contact-info {
  /* ... */
}
.contact-info strong {
  font-size: 1.1em;
}

/* 【新】按钮组的 div */
.contact-buttons {
  /* ... */
}

/* 【新】修改按钮样式 */
.edit-button {
  background-color: #ffc107; /* 黄色 */
  font-size: 0.8em;
  padding: 5px 10px;
}
.edit-button:hover {
  background-color: #e0a800;
}

/* 【新】取消按钮样式 */
.cancel-button {
  background-color: #6c757d; /* 灰色 */
}
.cancel-button:hover {
  background-color: #5a6268;
}

.delete-button {
  background-color: #dc3545; /* 红色 */
  font-size: 0.8em;
  padding: 5px 10px;
}
.delete-button:hover {
  background-color: #c82333; /* 深红色 */
}
</style>