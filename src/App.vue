<script setup>
import { ref, onMounted } from 'vue'
import TodoItem from './components/TodoItem.vue'

const todos = ref([])

const newTodo = ref('')

function addTodo() {
  if (newTodo.value.trim() === '') {
    return
  }

  todos.value.push({
    id: Date.now(),
    title: newTodo.value,
    completed: false
  })

  newTodo.value = ''

  saveTodos()
}

function deleteTodo(id) {
  todos.value = todos.value.filter(todo => todo.id !==id )

  saveTodos()
}

function updateTodo(updatedTodo) {
  const index = todos.value.findIndex(todo => todo.id === updatedTodo.id)

  if (index !== -1) {
    todos.value[index] = updatedTodo
  }

  saveTodos()
}

function saveTodos() {
 localStorage.setItem('todos', JSON.stringify(todos.value))
}

onMounted(() => {
  const savedTodos = localStorage.getItem('todos')

  if (savedTodos) {
    todos.value = JSON.parse(savedTodos)
  }
})

</script>


<template>
  <div class="container">

    <h1>My To-Do List</h1>

    <!-- Add Todo -->
    <div class="add-todo">
      <input
      v-model="newTodo"
      type="text"
      placeholder="Enter a task"
      @keyup.enter="addTodo"
      />

      <button @click="addTodo">
        Add
      </button>
    </div>

    <!-- Todo List -->
    <div class="todo-list">

      <TodoItem
      v-for="todo in todos"
      :key="todo.id"
      :todo="todo"
      @delete="deleteTodo"
      @update="updateTodo"
      />

    </div>

  </div>
</template>


<style>
.container {
  width: 500px;
  margin: 50px auto;
  font-family: Arial, sans-serif;
}


h1 {
  text-align: center;
}


.add-todo {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}


.add-todo input {
  flex: 1;
  padding: 10px;
}


button {
  padding: 10px 15px;
  cursor: pointer;
}


.todo-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
</style>