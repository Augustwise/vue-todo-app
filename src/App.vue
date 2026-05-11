<script setup>
import { ref, computed, onBeforeMount, watch } from 'vue'
import StatusFilter from './components/StatusFilter.vue'
import TodoItem from './components/TodoItem.vue'

const todos = ref([])
const title = ref('')
const errorMessage = ref('')
const status = ref('all')

function addTodo() {
  if (title.value.trim() === '') {
    errorMessage.value = 'Title should not be empty'
    return
  }
  todos.value.push({
    id: Date.now(),
    title: title.value,
    completed: false,
  })
  title.value = ''
}

const visibleTodos = computed(() => {
  if (status.value === 'active') {
    return todos.value.filter(todo => !todo.completed)
  }
  if (status.value === 'completed') {
    return todos.value.filter(todo => todo.completed)
  }
  return todos.value
})

const activeTodos = computed(() => todos.value.filter(todo => !todo.completed))

watch(
  todos,
  newTodos => {
    localStorage.setItem('todos', JSON.stringify(newTodos))
  },
  { deep: true }
)

onBeforeMount(() => {
  todos.value = JSON.parse(localStorage.getItem('todos')) || []
})
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos {{ todos.length }}</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <!-- this button should have `active` class only if all todos are completed -->
        <button
          v-if="activeTodos.length === 0"
          class="todoapp__toggle-all"
          :class="{ active: activeTodos.length === 0 }"
        ></button>

        <form @submit.prevent="addTodo">
          <input v-model="title" class="todoapp__new-todo" placeholder="What needs to be done?" />
        </form>
      </header>

      <section class="todoapp__main">
        <TodoItem
          v-for="todo of visibleTodos"
          :key="todo.id"
          :todo="todo"
          @toggle="todo.completed = !todo.completed"
          @remove="todos.splice(todos.indexOf(todo), 1)"
        />
      </section>

      <!-- Hide the footer if there are no todos -->
      <footer class="todoapp__footer">
        <!-- show the number of not caompleted todos -->
        <span class="todo-count">{{ activeTodos.length }} items left</span>

        <StatusFilter v-model:status="status" />

        <!-- this button should be disabled if there are no completed todos -->
        <button
          class="todoapp__clear-completed"
          :disabled="activeTodos.length === todos.length"
          @click="todos = activeTodos"
        >
          Clear completed
        </button>
      </footer>
    </div>

    <div
      class="notification is-danger is-light has-text-weight-normal"
      :class="{ hidden: !errorMessage }"
    >
      <button class="delete" @click="errorMessage = ''"></button>
      <!-- show only one message at a time -->
      {{ errorMessage }}
    </div>
  </div>
</template>
