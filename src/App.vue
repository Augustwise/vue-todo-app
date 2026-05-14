<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import StatusFilter from './components/StatusFilter.vue'
import TodoItem from './components/TodoItem.vue'
import { getTodos, createTodo, updateTodo, deleteTodo } from './api/todos'
import Message from './components/Message.vue'

const todos = ref([])
const title = ref('')
const errorMessage = ref('')
const status = ref('all')

const addTodo = async () => {
  if (!title.value) {
    errorMessage.value = 'Title should not be empty'
    return
  }
  try {
    const newTodo = await createTodo(title.value)
    todos.value.push(newTodo)
  } catch (error) {
    errorMessage.value = 'Unable to add a todo'
  } finally {
    title.value = ''
  }
}

const removeTodo = async todo => {
  try {
    await deleteTodo(todo.id)
    todos.value.splice(todos.value.indexOf(todo), 1)
  } catch (error) {
    errorMessage.value = 'Unable to delete a todo'
  }
}

const renameTodo = async (todo, title) => {
  try {
    const updatedTodo = await updateTodo({
      ...todo,
      title,
    })
    Object.assign(todo, updatedTodo)
  } catch (error) {
    errorMessage.value = 'Unable to update a todo'
  }
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

onMounted(async () => {
  try {
    todos.value = await getTodos()
  } catch (error) {
    errorMessage.value = 'Unable to load todos'
  }
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

      <TransitionGroup v-if="todos.length > 0" tag="section" name="todolist" class="todoapp__main">
        <TodoItem
          v-for="todo of visibleTodos"
          :key="todo.id"
          :todo="todo"
          @toggle="todo.completed = !todo.completed"
          @remove="removeTodo(todo)"
          @update="updatedTodo => renameTodo(todo, updatedTodo.title)"
        />
      </TransitionGroup>

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

<style scoped>
.todolist-enter-active,
.todolist-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}
.todolist-enter-from,
.todolist-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>
