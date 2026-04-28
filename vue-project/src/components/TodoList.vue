<script setup lang="ts">
import TodoItem from './TodoItem.vue';
import type { Todo } from '@/types/todo';

defineProps<{
    todos: Todo[]
}>()

const emit = defineEmits<{
    deleteTodo: [id: number]
    toggleTodo: [id: number]
    updateTodo: [payload:  { id: number, title: string}]
}>()
</script>
<template>
    <p v-if="todos.length === 0" class="empty">No todos found.</p>
    <ul>
        <TodoItem
        v-for="todo in todos"
        :key="todo.id"
        :todo="todo"
        @delete-todo="emit('deleteTodo', $event)"
        @toggle-todo="emit('toggleTodo', $event)"
        @update-todo="emit('updateTodo', $event)"
        />
    </ul>
</template>