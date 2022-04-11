<script setup>
import {ref, computed, watchEffect} from 'vue'
import './App.css'

const STORAGE_KEY = 'todo-list'

const filters = {
  all: todos => todos,
  active: todos => todos.filter(todo => !todo.completed),
  completed: todos => todos.filter(todo => todo.completed)
}

// 初始資料
const todos = ref(JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'))
const visibility = ref('all')
const editedTodo = ref()

// 資料計算
  // 未完成的數量
const remaining = computed(() => {
  return filters.active(todos.value).length
})
  // 過濾資料
const filteredTodos = computed(() => {
  return filters[visibility.value](todos.value)
})

// 路徑處理
window.addEventListener('hashchange', onHashChange)
onHashChange()

// 持續狀態
watchEffect(() => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
})

// 函式
  //新增 Todo
function addTodo(e){
  const value = e.target.value.trim()
  if(value) {
    todos.value.push({
      id: Date.now(),
      title: value,
      completed: false
    })
    e.target.value = ''
  }
}

  //刪除 Todo
function removeTodo(todo) {
  todos.value.splice(todos.value.indexOf(todo), 1)
}

  // 編輯 Todo
let beforeEditCache = ''
function editTodo(todo) {
  beforeEditCache = todo.title
  editedTodo.value = todo
}
    // 取消編輯
function cancelEdit(todo) {
  editedTodo.value = null
  todo.title = beforeEditCache
}
    // 完成編輯
function doneEdit(todo) {
  if (editedTodo.value) {
    editedTodo.value = null
    todo.title = todo.title.trim()
    if (!todo.title) {
      removeTodo(todo)
    }
  }
}

  //路徑變換
function onHashChange() {
  const route = window.location.hash.replace(/#\/?/, '')
  if(filters[route]){
    visibility.value = route
  } else {
    window.location.hash = ''
    visibility.value = 'all'
  }
}

  // 清除所有已完成 -> 把所有未完成的放到 todos
function removeCompleted() {
  todos.value = filters.active(todos.value)
}

console.log(todos.value)
console.log(window.location)
</script>

<template>
  <div class="container">
    <header class="header">
      <h1 class="title">TODO LIST</h1>
      <input class="todo-input" type="text" autofocus placeholder="What needs to be done?" @keyup.enter="addTodo">
    </header>
    <section class="main">
      <span class="tip" v-show="todos.length">Double click on the list to edit.</span>
      <ul class="lists">
        <li class="list" :key="todo.id" v-for="todo in filteredTodos">
          <div class="list-content">
            <input class="list-checkbox" type="checkbox" v-model="todo.completed">
            <label class="todo-title" @dblclick="editTodo(todo)" v-on:tap="editTodo(todo)">{{todo.title}}</label>
          </div>
          <button class="btn-delete" :title="'Delete'" @click="removeTodo(todo)">x</button>
          <input
            class="editTodo-input"
            v-if="todo === editedTodo"
            type="text"
            v-model="todo.title"
            @blur="doneEdit(todo)"
            @keyup.enter="doneEdit(todo)"
            @keyup.escape="cancelEdit(todo)"
            >
        </li>
      </ul>
    </section>
    <footer>
      <div class="items-left">
        <strong>{{ remaining }}</strong>
        <span>{{ remaining === 1 ? ' item' : ' items'}} left</span>
      </div>
      <div class="navbar">
        <ul class="nav">
          <li>
            <a :class="{selected: visibility === 'all'}" href="#/all">All</a>
          </li>
          <li>
            <a :class="{selected: visibility === 'active'}" href="#/active">Active</a>
          </li>
          <li>
            <a :class="{selected: visibility === 'completed'}" href="#/completed">Completed</a>
          </li>
        </ul>
        <button class="btn-clear-all" @click="removeCompleted" v-show="todos.length > remaining">Clear completed</button>
      </div>
    </footer>
  </div>
</template>