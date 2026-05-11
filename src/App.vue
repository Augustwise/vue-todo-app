<script setup>
import { ref, computed, onBeforeMount, watch } from "vue";

const todos = ref([]);
const title = ref("");
const errorMessage = ref("");
const status = ref("all");

function addTodo() {
  if (title.value.trim() === "") {
    errorMessage.value = "Title should not be empty";
    return;
  }
  todos.value.push({
    id: Date.now(),
    title: title.value,
    completed: false,
  });
  title.value = "";
}

const visibleTodos = computed(() => {
  if (status.value === "active") {
    return todos.value.filter(todo => !todo.completed);
  }
  if (status.value === "completed") {
    return todos.value.filter(todo => todo.completed);
  }
  return todos.value;
});

const activeTodos = computed(() => todos.value.filter(todo => !todo.completed));

watch(todos, newTodos => {
  localStorage.setItem("todos", JSON.stringify(newTodos));
}, { deep: true });

onBeforeMount(() => {
  todos.value = JSON.parse(localStorage.getItem("todos")) || [];
});
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
        :class="{ 'active': activeTodos.length === 0 }"
        ></button>

        <form @submit.prevent="addTodo">
          <input
            class="todoapp__new-todo"
            placeholder="What needs to be done?"
            v-model="title"
          />
        </form>
      </header>

      <section class="todoapp__main">
        <div
          v-for="todo, i of visibleTodos"
          class="todo"
          :class="{ completed: todo.completed }"
        >
          <label class="todo__status-label">
            <input
              type="checkbox"
              class="todo__status"
              v-model="todo.completed"
            />
          </label>

          <!-- show when todo is being edited -->
          <form v-if="false">
            <input
              class="todo__title-field"
              placeholder="Empty todo will be deleted"
            />
          </form>

          <template v-else>
            <span class="todo__title">{{ todo.title }}</span>
            <button 
            class="todo__remove" 
            @click="todos.splice(i, 1)">×</button>
          </template>

          <!-- add `is-active` class when todo being processed -->
          <div class="modal overlay" :class="{ 'is-active': false }">
            <div class="modal-background has-background-white-ter"></div>
            <div class="loader"></div>
          </div>
        </div>
      </section>

      <!-- Hide the footer if there are no todos -->
      <footer class="todoapp__footer">
        <!-- show the number of not caompleted todos -->
        <span class="todo-count">{{ activeTodos.length }} items left</span>

        <!-- Active link should have the 'selected' class -->
        <nav class="filter">
          <a href="#/" 
          class="filter__link" :class="{ 'selected': status === 'all' }" @click="status = 'all'"
          >All</a>
          <a href="#/active" class="filter__link" :class="{ 'selected': status === 'active' }" @click="status = 'active'">Active</a>
          <a href="#/completed" class="filter__link" :class="{ 'selected': status === 'completed' }" @click="status = 'completed'">Completed</a>
        </nav>

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

    <div class="notification is-danger is-light has-text-weight-normal" :class="{ 'hidden': !errorMessage }">
      <button class="delete" @click="errorMessage = ''"></button>
      <!-- show only one message at a time -->
      {{ errorMessage }}
    </div>
  </div>
</template>