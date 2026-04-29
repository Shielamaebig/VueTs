<script setup lang="ts">
import { ref, computed, reactive } from 'vue';
import TodoForm from './components/TodoForm.vue';
import TodoList from './components/TodoList.vue';
import type { Todo } from './types/todo';

const todos = ref<Todo[]>([]);
const filter = ref<'all' | 'active' | 'completed'>('all');

const count = ref(0)
const name = ref("Shiela")
const user = reactive({age: 28})

const addTodo = (title: string) => {
  const newTodo: Todo = {
    id: Date.now(),
    title,
    completed: false,
  }
  todos.value.push(newTodo);
}

const deleteTodo = (id: number) =>  {
  todos.value = todos.value.filter(todo => todo.id !== id);
}

const toggleTodo = (id: number) => {
  const todo = todos.value.find(todo => todo.id === id);
  if (todo) {
    todo.completed = !todo.completed;
  }
}
const updateTodo = (payload: { id: number, title: string}) => {
  const todo = todos.value.find(todo => todo.id === payload.id);
  if (todo) {
    todo.title = payload.title;
  }
}

const filteredTodos = computed(() => {
  if (filter.value === 'active') {
    return todos.value.filter(todo => !todo.completed)
  }
  if (filter.value === 'completed') {
    return todos.value.filter(todo => todo.completed)
  }
  return todos.value;
})

</script>
<template>
  <main class="app">
    <h1> Vue TypeScript Todo App</h1>
    <TodoForm @add-todo="addTodo" />
    <div class="filters">
      <button @click="filter = 'all'">ALL</button>
      <button @click="filter = 'active'">Active</button>
      <button @click="filter = 'completed'">Completed</button>
    </div>
    <TodoList :todos="filteredTodos" 
    @delete-todo="deleteTodo" 
    @toggle-todo="toggleTodo"
    @update-todo="updateTodo"/>
  </main>

  <button @click="count++">{{ count }}</button>
</template>

<style scoped></style>
