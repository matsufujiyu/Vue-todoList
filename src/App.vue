<script setup>
import { ref, useTemplateRef, watchEffect,onMounted } from 'vue'
const tasks = ref([])
const userInput = ref('')
// 入力欄にフォーカスを当てる
const userInputEl=useTemplateRef<HTMLInputElement>('userInputRef')

function addTask() {
  if (userInput.value !== '') {
    tasks.value = [
      {
        id: Date.now(),
        name: userInput.value,
        status: ''  // 初期ステータスは空
      },
      ...tasks.value  // 既存のタスクを後ろに追加
    ]
    userInput.value = ''
    userInputEl.value?.focus()
    sortTasks()
  }
}
onMounted(() => {
  userInputEl.value?.focus()  // 最初にinput1にフォーカスを当てる
})



// 編集モードを切り替えるための関数
function editTaskName(taskId) {
  const task = tasks.value.find(task => task.id === taskId)
  if (task) {
    task.isEditing = true
  }
}

// 編集を終了し、タスク名を更新する関数
function saveTaskName(taskId) {
  const task = tasks.value.find(task => task.id === taskId)
  if (task) {
    task.isEditing = false
    
  }
}

// タスクをステータス順に並び替える関数
function sortTasks() {
  tasks.value.sort((a, b) => {
    const order = { '': 1, 'in-progress': 0, 'completed': 2 }
    return order[a.status] - order[b.status]
  })
}
// タスクを削除する関数
function removeTask(taskId) {
  // タスクを削除する
  tasks.value = tasks.value.filter(task => task.id !== taskId)
}

watchEffect(() => {
  console.log('Tasks updated:', tasks.value)
})

// ステータスに応じて適切なクラスを返す関数
function getStatusClass(status) {
  switch (status) {
    case 'in-progress':
      return 'in-progress';
    case 'completed':
      return 'completed';
    default:
      return 'pending'; // 未着手
  }
}



</script>

<template>
      <h1>TodoList</h1>
      <p>リストの追加</p>
  <input v-model="userInput" type="text"  ref='userInputRef'/>
  <button @click="addTask">追加</button>

  <p>リスト</p>
  <ul>
  <li v-for="task in tasks" :key="task.id" :class="getStatusClass(task.status)" class="task-item">
  <!-- 編集モードを切り替える -->
  <span v-if="!task.isEditing" @click="editTaskName(task.id)" class="task-name">
        {{ task.name }}
      </span>
      
      <span
        v-if="task.isEditing"
        class="task-name-edit"
        contenteditable="true"
        @keydown.enter="saveTaskName(task.id)"  
        @input="task.name = $event.target.innerText" 
      >
        {{ task.name }}
      </span>

      <div class="task-status-container">
    <select v-model="task.status" class="task-status" @change="sortTasks">
        <option value="">未着手</option>
        <option value="in-progress">進行中</option>
        <option value="completed">完了</option>
      </select>
      <button @click="removeTask(task.id)" class="task-remove-btn">削除</button>
    </div>
    </li>
</ul>
 
 </template>

<style >
body {
  background-color: #f7e0be; /* クリーム色 */
  font-family: Arial, sans-serif;
}

ul{
  padding-left:0;
}

li{
  list-style:none;
}

.task-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px; /* タスク間のスペース */
  padding: 10px;
  border-radius: 5px;
  width: 100%;
}
.task-name-edit {
  margin-right: 10px;
  outline: none;
  border-bottom: 1px dashed #aaa;
  min-width: 100px; /* タスク名の最小幅を指定 */
  display: inline-block; /* インライン表示 */
}

.task-name {
  margin-right: 10px; /* name と status の間のスペース */
  cursor: pointer;
}

.task-name-edit {
  margin-right: 10px;
  outline: none;
  border-bottom: 1px dashed #aaa;
  min-width: 100px; /* タスク名の最小幅を指定 */
  display: inline-block; /* インライン表示 */
}

.task-status-container {
  display: flex;
  align-items: center; /* セレクトボックスと削除ボタンを縦に揃える */
}

.task-status {
  margin-right: 10px; /* status と削除ボタンの間のスペース */
}

.task-remove-btn {
  margin-left: 10px; /* 削除ボタンと他の要素の間のスペース */
}

/* ステータスに応じた背景色 */
.task-item.pending {
  background-color: rgb(183, 216, 228); /* 未着手 */
}

.task-item.in-progress {
  background-color: rgb(190, 230, 190); /* 進行中 */
}

.task-item.completed {
  background-color: rgb(252, 201, 210); /* 完了 */
}
</style>
