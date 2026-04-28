<script setup lang="ts">
import { ref } from 'vue';

const title = ref('')

const emit = defineEmits<{
    addTodo: [title: string]
}>()

const handleSubmit = () => {
    const trimmedTitle = title.value.trim()

    if (!trimmedTitle) return

    emit('addTodo', trimmedTitle)

    title.value = ''
}
</script>

<template>
    <form @submit.prevent="handleSubmit">
        <input v-model="title" type="text" placeholder="Add a todo.."/>
        <button type="submit">Add</button>
    </form>
</template>