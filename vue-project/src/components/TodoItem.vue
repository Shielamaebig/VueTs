<script setup lang="ts">
import { ref } from 'vue';
import type { Todo } from '@/types/todo';

const props = defineProps<{
    todo: Todo
}>()

const emit = defineEmits<{
    deleteTodo: [id: number]
    toggleTodo: [id: number]
    updateTodo: [payload:  { id: number, title: string}]
}>()

const isEditing = ref(false)
const editedTitle = ref(props.todo.title)

const startEditing = () => {
    isEditing.value = true
    editedTitle.value = props.todo.title
}

const cancelEditing = () => {
    isEditing.value = false
    editedTitle.value = props.todo.title
}

const saveEdit = () => {
    const trimmedTitle = editedTitle.value.trim()

    if (!trimmedTitle) return

    emit('updateTodo', {
        id: props.todo.id,
        title: trimmedTitle
    })

    isEditing.value = false
}

</script>

<template>
    <li class="todo-item">
        <template v-if="isEditing">
            <input v-model="editedTitle" 
            @keyup.enter="saveEdit"
            @key.esc="cancelEditing"/>

            <button @click="saveEdit">Save</button>
            <button @click="cancelEditing">Cancel</button>
        </template>
        <template v-else>
            <input type="checkbox" :checked="todo.completed"
            @change="emit('toggleTodo', todo.id)"/>

            <span :class="{completed: todo.completed}">
                {{ todo.title }}
            </span>
            <button @click="startEditing">
                Edit
            </button>
            <button @click="emit('deleteTodo', todo.id)">
                Delete
            </button>
        </template>
    </li>
</template>