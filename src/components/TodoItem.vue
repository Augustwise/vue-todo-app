<script setup>
defineProps({
  todo: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['remove', 'toggle'])
</script>

<template>
  <div class="todo" :class="{ completed: todo.completed }">
    <label class="todo__status-label">
      <input
        :checked="todo.completed"
        type="checkbox"
        class="todo__status"
        @change="emit('toggle')"
      />
    </label>

    <!-- show when todo is being edited -->
    <form v-if="false">
      <input class="todo__title-field" placeholder="Empty todo will be deleted" />
    </form>

    <template v-else>
      <span class="todo__title">{{ todo.title }}</span>
      <button class="todo__remove" @click="emit('remove')">×</button>
    </template>

    <!-- add `is-active` class when todo being processed -->
    <div class="modal overlay" :class="{ 'is-active': false }">
      <div class="modal-background has-background-white-ter"></div>
      <div class="loader"></div>
    </div>
  </div>
</template>
