<script setup>
defineProps({
  status: {
    type: String,
    required: true,
  },
})

const emit = defineEmits(['update:status'])
</script>

<template>
  <nav class="filter">
    <a
      href="#/"
      class="filter__link"
      :class="{ selected: status === 'all' }"
      @click="emit('update:status', 'all')"
      >All</a
    >
    <a
      href="#/active"
      class="filter__link"
      :class="{ selected: status === 'active' }"
      @click="emit('update:status', 'active')"
      >Active</a
    >
    <a
      href="#/completed"
      class="filter__link"
      :class="{ selected: status === 'completed' }"
      @click="emit('update:status', 'completed')"
      >Completed</a
    >
  </nav>
</template>
