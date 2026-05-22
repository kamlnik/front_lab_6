<template>
  <div class="todo-app">
    <h1> ToDo List </h1>
    <div class="add-todo">
      <input 
        v-model="newTodoText" 
        @keyup.enter="addTodo" 
        placeholder="Новая задача..." 
      />
      <button @click="addTodo" :disabled="isLoading">+ Добавить</button>
    </div>
    <div v-if="isLoading" class="loading">Загрузка...</div>
    <ul class="todo-list">
      <li 
        v-for="todo in todos" 
        :key="todo.id" 
        :class="{ 'completed': todo.completed }"
      >
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleTodo(todo)"
          :disabled="isLoading"
        />
        <span>{{ todo.title }}</span>
        <button @click="openDeletePopup(todo.id)" :disabled="isLoading">🗑️</button>
      </li>
    </ul>
    <p v-if="!isLoading && todos.length === 0">Нет задач. Добавьте первую!</p>

    <!--  подтверждения удаления -->
    <Popup v-model="showDeletePopup">
      <p>? Удалить задачу <strong>{{ getTodoTitleForDelete }}</strong>?</p>
      <div style="display: flex; gap: 10px; justify-content: flex-end; margin-top: 16px;">
        <button @click="confirmDelete" :disabled="isLoading">Да, удалить</button>
        <button @click="showDeletePopup = false">Отмена</button>
      </div>
    </Popup>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import Popup from './components/Popup.vue'

// конфиг API 
const API_BASE = 'https://jsonplaceholder.typicode.com/todos'
const USER_ID = 1
const TIMEOUT_MS = 5000

// переменные данные
const todos = ref([])
const newTodoText = ref('')
const isLoading = ref(false)

// Состояния для подтверджения удаления 
const showDeletePopup = ref(false)
const todoToDeleteId = ref(null)

// вычисление названия удаляемого таска
const getTodoTitleForDelete = computed(() => {
  const todo = todos.value.find(t => t.id === todoToDeleteId.value)
  return todo ? todo.title : ''
})


// fetch с таймаутом
const fetchWithTimeout = (url, options = {}, timeout = TIMEOUT_MS) => {
  return Promise.race([
    fetch(url, options),
    new Promise((_, reject) => setTimeout(() => reject(new Error('Timeout')), timeout))
  ])
}

// Сохранение в localStorage
const saveToLocalStorage = () => {
  localStorage.setItem('todos', JSON.stringify(todos.value))
}

// Загрузка начальных данных
const loadTodos = async () => {
  isLoading.value = true
  try {
    const stored = localStorage.getItem('todos')
    if (stored && JSON.parse(stored).length > 0) {
      todos.value = JSON.parse(stored)
      console.log('Загружено из localStorage')
      return
    }
    // загрузка из API 
    console.log('Загружаем из API')
    const response = await fetchWithTimeout(`${API_BASE}?userId=${USER_ID}`)
    if (!response.ok) throw new Error('HTTP error')
    const data = await response.json()
    todos.value = data.slice(0, 10).map(todo => ({
      id: todo.id,
      title: todo.title,
      completed: todo.completed
    }))
    saveToLocalStorage()
  } catch (error) {
    console.error('Ошибка загрузки:', error)

    if (!localStorage.getItem('todos')) {
      todos.value = [
        { id: 1, title: 'Изучить Vue 3', completed: false },
        { id: 2, title: 'Сделать ToDo', completed: true }
      ]
      saveToLocalStorage()
      alert('Не удалось загрузить данные. Используются примеры задач.')
    }
  } finally {
    isLoading.value = false
  }
}

// Добавление задачи
const addTodo = async () => {
  const text = newTodoText.value.trim()
  if (text === '' || isLoading.value) return

  isLoading.value = true
  try {
    const response = await fetchWithTimeout(API_BASE, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        title: text,
        completed: false,
        userId: USER_ID
      })
    })
    if (!response.ok) throw new Error('Ошибка добавления')
    const newTodoFromApi = await response.json()
    todos.value.push({
      id: newTodoFromApi.id,
      title: text,
      completed: false
    })
    saveToLocalStorage()
    newTodoText.value = ''
  } catch (error) {
    alert('Не удалось добавить задачу: ' + error.message)
  } finally {
    isLoading.value = false
  }
}

// изменение статуса задачи
const toggleTodo = async (todo) => {
  const newCompleted = !todo.completed
  isLoading.value = true
  try {
    const response = await fetchWithTimeout(`${API_BASE}/${todo.id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ completed: newCompleted })
    })
    if (!response.ok) throw new Error('Ошибка обновления')
    todo.completed = newCompleted
    saveToLocalStorage()
  } catch (error) {
    alert('Не удалось изменить статус: ' + error.message)
  } finally {
    isLoading.value = false
  }
}

// Удаление задачи 
const deleteTodo = async (id) => {
  if (isLoading.value) return
  isLoading.value = true
  try {
    const response = await fetchWithTimeout(`${API_BASE}/${id}`, {
      method: 'DELETE'
    })
    if (!response.ok) throw new Error('Ошибка удаления')
    todos.value = todos.value.filter(t => t.id !== id)
    saveToLocalStorage()
  } catch (error) {
    alert('Не удалось удалить задачу: ' + error.message)
  } finally {
    isLoading.value = false
  }
}

// Открыть подтверждение удаления
const openDeletePopup = (id) => {
  todoToDeleteId.value = id
  showDeletePopup.value = true
}

// Подтверждение удаления
const confirmDelete = async () => {
  if (todoToDeleteId.value !== null) {
    await deleteTodo(todoToDeleteId.value)
    showDeletePopup.value = false
    todoToDeleteId.value = null
  }
}

// Загрузка данных при старте
onMounted(() => {
  loadTodos()
})
</script>

<style scoped>
.todo-app {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}
.add-todo {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
input[type="text"] {
  flex: 1;
  padding: 8px;
}
button {
  cursor: pointer;
  padding: 8px 12px;
}
button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
.todo-list {
  list-style: none;
  padding: 0;
}
.todo-list li {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px;
  border-bottom: 1px solid #eee;
}
.todo-list li.completed span {
  text-decoration: line-through;
  color: gray;
}
button:last-child {
  margin-left: auto;
  background: #5372e4;
  color: white;
  border: none;
  border-radius: 4px;
}
.loading {
  text-align: center;
  color: #666;
  margin: 10px 0;
}
</style>