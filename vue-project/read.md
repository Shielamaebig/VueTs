# Vue + TypeScript Todo App Learning Guide

This guide explains the code in this project step by step. You are learning two things at the same time:

- Vue: the frontend framework that builds the user interface.
- TypeScript: JavaScript with types, which helps catch mistakes earlier.

This project is a Todo app. It lets you add todos, filter them, mark them completed, edit them, and delete them.

## 1. Project Structure

Here are the important files:

```text
src/
  main.ts
  App.vue
  style.css
  types/
    todo.ts
  components/
    TodoForm.vue
    TodoList.vue
    TodoItem.vue
```

What each file does:

- `src/main.ts` starts the Vue app.
- `src/App.vue` is the main component. It owns the todo data.
- `src/components/TodoForm.vue` contains the form for adding a todo.
- `src/components/TodoList.vue` displays a list of todos.
- `src/components/TodoItem.vue` displays one todo item.
- `src/types/todo.ts` defines the TypeScript shape of a todo.
- `src/style.css` contains global styles for the app.

There are also project setup files:

- `package.json` lists scripts and installed packages.
- `vite.config.ts` configures Vite, the development tool.
- `tsconfig.app.json` configures TypeScript for the app.

## 2. What Is a Vue Component?

A Vue component is one piece of the user interface.

For example:

- `TodoForm.vue` is the add-todo form.
- `TodoList.vue` is the todo list.
- `TodoItem.vue` is one row in the list.

Most `.vue` files have these sections:

```vue
<script setup lang="ts">
// TypeScript and Vue logic goes here
</script>

<template>
  <!-- HTML-like UI goes here -->
</template>

<style scoped>
/* CSS for this component goes here */
</style>
```

In this project, the components use:

```vue
<script setup lang="ts">
```

This means:

- `script setup` uses Vue's simpler Composition API syntax.
- `lang="ts"` means the code inside is TypeScript.

## 3. Starting the App: `src/main.ts`

```ts
import { createApp } from "vue";
import App from "./App.vue";
import "./style.css";

createApp(App).mount("#app");
```

Line by line:

```ts
import { createApp } from "vue";
```

This imports Vue's `createApp` function.

```ts
import App from "./App.vue";
```

This imports the main Vue component.

```ts
import "./style.css";
```

This loads global CSS for the whole app.

```ts
createApp(App).mount("#app");
```

This creates the Vue app and places it inside the HTML element with `id="app"`.

That `#app` element comes from `index.html`.

## 4. The Todo Type: `src/types/todo.ts`

```ts
export type Todo = {
  id: number;
  title: string;
  completed: boolean;
};
```

This defines the shape of a todo object.

A todo must have:

- `id`: a number
- `title`: a string
- `completed`: a boolean, meaning `true` or `false`

Example todo:

```ts
const exampleTodo: Todo = {
  id: 1,
  title: "Learn Vue",
  completed: false,
};
```

Why this is useful:

TypeScript can warn you if you create a todo incorrectly.

For example, this would be wrong:

```ts
const badTodo: Todo = {
  id: "abc",
  title: "Learn Vue",
  completed: false,
};
```

The `id` is wrong because `"abc"` is a string, but the `Todo` type says `id` must be a number.

## 5. Main App Logic: `src/App.vue`

`App.vue` is the parent component. It stores the todos and controls the main behavior.

### Imports

```ts
import { ref, computed } from 'vue';
import TodoForm from './components/TodoForm.vue';
import TodoList from './components/TodoList.vue';
import type { Todo } from './types/todo';
```

This imports:

- `ref`: creates reactive values.
- `computed`: creates values based on other reactive values.
- `TodoForm`: the form component.
- `TodoList`: the list component.
- `Todo`: the TypeScript type for todos.

Notice this line:

```ts
import type { Todo } from './types/todo';
```

`import type` means you are only importing a TypeScript type, not real JavaScript code that runs in the browser.

### Reactive State with `ref`

```ts
const todos = ref<Todo[]>([]);
const filter = ref<'all' | 'active' | 'completed'>('all');
```

`ref` creates reactive data.

Reactive means: when the value changes, Vue updates the screen automatically.

This line:

```ts
const todos = ref<Todo[]>([]);
```

means:

- `todos` is reactive.
- It starts as an empty array.
- It must contain `Todo` objects.

This line:

```ts
const filter = ref<'all' | 'active' | 'completed'>('all');
```

uses a TypeScript union type.

It means `filter` can only be one of these strings:

- `'all'`
- `'active'`
- `'completed'`

This protects you from typing an invalid filter like `'done'`.

### Important `.value` Rule

When using `ref` inside `<script setup>`, you access the value with `.value`.

Example:

```ts
todos.value.push(newTodo);
filter.value === 'active';
```

Inside the template, Vue unwraps refs automatically, so you write:

```vue
<button @click="filter = 'all'">ALL</button>
```

You do not need `filter.value` inside the template.

## 6. Adding a Todo

```ts
const addTodo = (title: string) => {
  const newTodo: Todo = {
    id: Date.now(),
    title,
    completed: false,
  }
  todos.value.push(newTodo);
}
```

This function receives a todo title.

```ts
title: string
```

This means the function expects `title` to be a string.

Then it creates a new todo:

```ts
const newTodo: Todo = {
  id: Date.now(),
  title,
  completed: false,
}
```

`Date.now()` creates a number based on the current time. This is used as the todo id.

Then the todo is added to the array:

```ts
todos.value.push(newTodo);
```

Because `todos` is reactive, Vue updates the screen.

## 7. Deleting a Todo

```ts
const deleteTodo = (id: number) =>  {
  todos.value = todos.value.filter(todo => todo.id !== id);
}
```

This function receives an id.

```ts
id: number
```

It keeps every todo except the one with the matching id.

```ts
todo.id !== id
```

This means: keep the todo if its id is not the id we want to delete.

## 8. Toggling a Todo

```ts
const toggleTodo = (id: number) => {
  const todo = todos.value.find(todo => todo.id === id);
  if (todo) {
    todo.completed = !todo.completed;
  }
}
```

This finds the todo with the matching id.

```ts
const todo = todos.value.find(todo => todo.id === id);
```

Then it checks if the todo exists:

```ts
if (todo) {
```

This is important because `.find()` might return `undefined`.

Then it flips the completed value:

```ts
todo.completed = !todo.completed;
```

If `completed` was `false`, it becomes `true`.

If `completed` was `true`, it becomes `false`.

## 9. Updating a Todo

```ts
const updateTodo = (payload: { id: number, title: string}) => {
  const todo = todos.value.find(todo => todo.id === payload.id);
  if (todo) {
    todo.title = payload.title;
  }
}
```

This function receives a payload object.

The payload must look like this:

```ts
{
  id: number,
  title: string
}
```

The function finds the todo by id and replaces its title.

## 10. Filtering Todos with `computed`

```ts
const filteredTodos = computed(() => {
  if (filter.value === 'active') {
    return todos.value.filter(todo => !todo.completed)
  }
  if (filter.value === 'completed') {
    return todos.value.filter(todo => todo.completed)
  }
  return todos.value;
})
```

`computed` creates a value based on other reactive values.

Here, `filteredTodos` depends on:

- `todos`
- `filter`

If either one changes, Vue recalculates `filteredTodos`.

The logic:

- If the filter is `'active'`, show only todos that are not completed.
- If the filter is `'completed'`, show only completed todos.
- Otherwise, show all todos.

## 11. App Template

```vue
<template>
  <main class="app">
    <h1> Vue TypeScript Todo App</h1>
    <TodoForm @add-todo="addTodo" />
    <div class="filters">
      <button @click="filter = 'all'">ALL</button>
      <button @click="filter = 'active'">Active</button>
      <button @click="filter = 'completed'">Completed</button>
    </div>
    <TodoList
      :todos="filteredTodos"
      @delete-todo="deleteTodo"
      @toggle-todo="toggleTodo"
      @update-todo="updateTodo"
    />
  </main>
</template>
```

This renders the main UI.

### Listening to Events

```vue
<TodoForm @add-todo="addTodo" />
```

`@add-todo` means: listen for an event called `add-todo`.

When `TodoForm` emits that event, run the `addTodo` function.

### Click Events

```vue
<button @click="filter = 'active'">Active</button>
```

`@click` runs code when the button is clicked.

This changes the current filter.

### Passing Props

```vue
<TodoList :todos="filteredTodos" />
```

`:todos` passes data into the `TodoList` component.

This is called a prop.

A prop is data sent from a parent component to a child component.

## 12. Add Todo Form: `src/components/TodoForm.vue`

This component lets the user type a new todo and submit it.

### Local Form State

```ts
import { ref } from 'vue';

const title = ref('')
```

`title` stores what the user types into the input.

It starts as an empty string.

### Typed Emits

```ts
const emit = defineEmits<{
    addTodo: [title: string]
}>()
```

`defineEmits` defines events this component can send to its parent.

This says:

- The component can emit an event named `addTodo`.
- That event sends one value: `title`.
- `title` must be a string.

In the template, Vue uses kebab-case event names:

```vue
@add-todo="addTodo"
```

In the script, the event is written as camelCase:

```ts
emit('addTodo', trimmedTitle)
```

Vue connects these two styles.

### Submit Handler

```ts
const handleSubmit = () => {
    const trimmedTitle = title.value.trim()

    if (!trimmedTitle) return

    emit('addTodo', trimmedTitle)

    title.value = ''
}
```

This function:

1. Removes extra spaces from the title.
2. Stops if the title is empty.
3. Sends the title to the parent component.
4. Clears the input.

### Form Template and `v-model`

```vue
<template>
    <form @submit.prevent="handleSubmit">
        <input v-model="title" type="text" placeholder="Add a todo.."/>
        <button type="submit">Add</button>
    </form>
</template>
```

Important parts:

```vue
@submit.prevent="handleSubmit"
```

This runs `handleSubmit` when the form is submitted.

`.prevent` stops the browser from refreshing the page.

```vue
v-model="title"
```

`v-model` connects the input to the `title` ref.

When the user types, `title` updates.

When `title` changes in code, the input updates.

This is called two-way binding.

## 13. Todo List: `src/components/TodoList.vue`

This component receives todos and renders one `TodoItem` for each todo.

### Imports

```ts
import TodoItem from './TodoItem.vue';
import type { Todo } from '@/types/todo';
```

This imports the child component and the `Todo` type.

The `@` symbol means `src`.

So this:

```ts
@/types/todo
```

means:

```ts
src/types/todo
```

This works because both `vite.config.ts` and `tsconfig.app.json` define the alias.

### Props

```ts
defineProps<{
    todos: Todo[]
}>()
```

This says `TodoList` expects a prop named `todos`.

`todos` must be an array of `Todo` objects.

### Emits

```ts
const emit = defineEmits<{
    deleteTodo: [id: number]
    toggleTodo: [id: number]
    updateTodo: [payload:  { id: number, title: string}]
}>()
```

This component can emit three events:

- `deleteTodo`, with a number id
- `toggleTodo`, with a number id
- `updateTodo`, with an object containing id and title

### Empty State with `v-if`

```vue
<p v-if="todos.length === 0" class="empty">No todos found.</p>
```

`v-if` conditionally shows something.

This paragraph appears only when there are no todos.

### Looping with `v-for`

```vue
<TodoItem
  v-for="todo in todos"
  :key="todo.id"
  :todo="todo"
  @delete-todo="emit('deleteTodo', $event)"
  @toggle-todo="emit('toggleTodo', $event)"
  @update-todo="emit('updateTodo', $event)"
/>
```

`v-for` repeats a component for each todo.

```vue
v-for="todo in todos"
```

This means: for every todo in the todos array, create one `TodoItem`.

```vue
:key="todo.id"
```

The key helps Vue track each item efficiently.

```vue
:todo="todo"
```

This passes the todo into `TodoItem` as a prop.

```vue
@delete-todo="emit('deleteTodo', $event)"
```

This listens for a delete event from `TodoItem`, then sends it upward to `App.vue`.

`$event` is the value sent by the child event.

## 14. Todo Item: `src/components/TodoItem.vue`

This component displays and edits one todo.

### Props

```ts
const props = defineProps<{
    todo: Todo
}>()
```

This component expects one prop:

- `todo`, which must be a `Todo`

The code stores the props in a variable because it needs to use `props.todo` inside the script.

### Emits

```ts
const emit = defineEmits<{
    deleteTodo: [id: number]
    toggleTodo: [id: number]
    updateTodo: [payload:  { id: number, title: string}]
}>()
```

This component can tell its parent:

- delete this todo
- toggle this todo
- update this todo

It does not directly change the main todo array. It sends events upward.

### Editing State

```ts
const isEditing = ref(false)
const editedTitle = ref(props.todo.title)
```

`isEditing` tracks whether the item is in edit mode.

`editedTitle` stores the temporary title while editing.

### Start Editing

```ts
const startEditing = () => {
    isEditing.value = true
    editedTitle.value = props.todo.title
}
```

This turns on edit mode and copies the current todo title into the input.

### Cancel Editing

```ts
const cancelEditing = () => {
    isEditing.value = false
    editedTitle.value = props.todo.title
}
```

This exits edit mode and resets the input back to the original title.

### Save Edit

```ts
const saveEdit = () => {
    const trimmedTitle = editedTitle.value.trim()

    if (!trimmedTitle) return

    emit('updateTodo', {
        id: props.todo.id,
        title: trimmedTitle
    })

    isEditing.value = false
}
```

This:

1. Removes extra spaces.
2. Stops if the title is empty.
3. Emits `updateTodo` with the todo id and new title.
4. Exits edit mode.

### Conditional Template with `v-if` and `v-else`

```vue
<template v-if="isEditing">
```

This part shows when the item is being edited.

```vue
<template v-else>
```

This part shows when the item is not being edited.

### Edit Mode Template

```vue
<input
  v-model="editedTitle"
  @keyup.enter="saveEdit"
  @key.esc="cancelEditing"
/>

<button @click="saveEdit">Save</button>
<button @click="cancelEditing">Cancel</button>
```

`v-model` connects the input to `editedTitle`.

```vue
@keyup.enter="saveEdit"
```

This saves when the user presses Enter.

Learning note: the current code has:

```vue
@key.esc="cancelEditing"
```

For Vue keyboard events, this is usually written as:

```vue
@keyup.esc="cancelEditing"
```

This guide explains the current code, but that is a useful detail to remember as you learn.

### Normal Mode Template

```vue
<input
  type="checkbox"
  :checked="todo.completed"
  @change="emit('toggleTodo', todo.id)"
/>
```

This checkbox shows whether the todo is completed.

```vue
:checked="todo.completed"
```

The `:` means dynamic binding. The checkbox gets its checked value from `todo.completed`.

```vue
@change="emit('toggleTodo', todo.id)"
```

When the checkbox changes, the component emits `toggleTodo`.

### Class Binding

```vue
<span :class="{completed: todo.completed}">
    {{ todo.title }}
</span>
```

This displays the todo title.

```vue
{{ todo.title }}
```

Double curly braces show JavaScript values in the template.

This part:

```vue
:class="{completed: todo.completed}"
```

means:

- Add the `completed` CSS class if `todo.completed` is true.
- Do not add it if `todo.completed` is false.

In `style.css`, the class looks like this:

```css
.completed {
  text-decoration: line-through;
  color: gray;
}
```

So completed todos get crossed out.

## 15. Data Flow in This App

Vue apps usually have data flowing down and events flowing up.

In this project:

```text
App.vue
  owns the todos array
  passes todos down to TodoList

TodoList.vue
  receives todos
  passes each todo down to TodoItem

TodoItem.vue
  receives one todo
  emits events upward when the user edits, toggles, or deletes

TodoForm.vue
  emits an event upward when the user adds a todo
```

### Add Todo Flow

```text
User types in TodoForm
TodoForm emits addTodo
App.vue runs addTodo
App.vue updates todos
Vue updates the screen
```

### Delete Todo Flow

```text
User clicks Delete in TodoItem
TodoItem emits deleteTodo
TodoList forwards deleteTodo
App.vue runs deleteTodo
App.vue updates todos
Vue updates the screen
```

### Toggle Todo Flow

```text
User clicks checkbox in TodoItem
TodoItem emits toggleTodo
TodoList forwards toggleTodo
App.vue runs toggleTodo
App.vue updates the todo
Vue updates the screen
```

### Update Todo Flow

```text
User edits title in TodoItem
TodoItem emits updateTodo with id and title
TodoList forwards updateTodo
App.vue runs updateTodo
App.vue updates the todo title
Vue updates the screen
```

## 16. Global Styles: `src/style.css`

This file styles the app.

Example:

```css
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #f4f4f5;
}
```

This styles the whole page.

```css
.app {
  max-width: 640px;
  margin: 40px auto;
  padding: 24px;
  background: white;
  border-radius: 12px;
}
```

This styles the main app box.

```css
.todo-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px;
  border-bottom: 1px solid #ddd;
}
```

This makes each todo row line up horizontally.

## 17. Scoped Styles vs Global Styles

Some Vue components can have:

```vue
<style scoped>
</style>
```

`scoped` means the CSS only applies to that component.

In this project, `App.vue` has:

```vue
<style scoped></style>
```

It is empty, so it does not currently add styles.

The real styling is in `src/style.css`, which is global.

Global means it can affect the whole app.

## 18. Config Learning Notes

### `package.json`

```json
"scripts": {
  "dev": "vite",
  "build": "run-p type-check \"build-only {@}\" --",
  "preview": "vite preview",
  "build-only": "vite build",
  "type-check": "vue-tsc --build"
}
```

Useful commands:

```sh
npm run dev
```

Starts the development server.

```sh
npm run build
```

Type-checks and builds the app for production.

### `vite.config.ts`

```ts
resolve: {
  alias: {
    '@': fileURLToPath(new URL('./src', import.meta.url))
  },
},
```

This makes `@` point to the `src` folder.

That is why this works:

```ts
import type { Todo } from '@/types/todo';
```

### `tsconfig.app.json`

```json
"paths": {
  "@/*": ["./src/*"]
}
```

This tells TypeScript about the same alias.

Vite needs the alias to run the app.

TypeScript needs the alias to understand the code in your editor and during type-checking.

## 19. Vue Syntax Cheat Sheet

```vue
@click="doSomething"
```

Listen for a click event.

```vue
@submit.prevent="handleSubmit"
```

Listen for submit and prevent page refresh.

```vue
v-model="title"
```

Two-way bind an input to a reactive value.

```vue
v-if="todos.length === 0"
```

Show something only if the condition is true.

```vue
v-for="todo in todos"
```

Loop over an array.

```vue
:todo="todo"
```

Pass a dynamic value as a prop.

```vue
:class="{ completed: todo.completed }"
```

Add a CSS class only when a condition is true.

```vue
{{ todo.title }}
```

Display a value in the template.

## 20. TypeScript Cheat Sheet

```ts
const title = ref('')
```

TypeScript understands this is a string ref.

```ts
const todos = ref<Todo[]>([])
```

This is a reactive array of `Todo` objects.

```ts
const filter = ref<'all' | 'active' | 'completed'>('all')
```

This can only be one of three string values.

```ts
const addTodo = (title: string) => {}
```

The function parameter must be a string.

```ts
const updateTodo = (payload: { id: number, title: string }) => {}
```

The function parameter must be an object with `id` and `title`.

```ts
import type { Todo } from '@/types/todo'
```

Import only the TypeScript type.

## 21. Glossary

Component: A reusable piece of UI.

Reactive: Data that automatically updates the screen when it changes.

Ref: A Vue wrapper for reactive data. In script code, use `.value`.

Computed: A value calculated from reactive data.

Prop: Data passed from a parent component to a child component.

Emit: An event sent from a child component to a parent component.

Template: The HTML-like part of a Vue component.

Directive: Special Vue syntax like `v-if`, `v-for`, and `v-model`.

Type: A TypeScript rule that describes what shape a value should have.

Union type: A type that allows only certain values, like `'all' | 'active' | 'completed'`.

Payload: Data sent with an event or function call.

Alias: A shortcut for an import path. In this app, `@` means `src`.

## 22. Big Picture

The most important idea in this app is:

```text
App.vue owns the data.
Child components display data.
Child components emit events when something happens.
App.vue updates the data.
Vue updates the screen.
```

Once you understand that flow, Vue becomes much easier to learn.

## 23. Your Actual Code Explained More Slowly

This section walks through your Todo app code in a more direct way. Read this while looking at the files in `src`.

## 24. `main.ts` Explained

Your code:

```ts
import { createApp } from "vue";
import App from "./App.vue";
import "./style.css";

createApp(App).mount("#app");
```

Explanation:

```ts
import { createApp } from "vue";
```

This gets the `createApp` function from Vue. You need this function to start a Vue application.

```ts
import App from "./App.vue";
```

This imports your main component. Think of `App.vue` as the main container of your whole app.

```ts
import "./style.css";
```

This loads your CSS file. Because it is imported in `main.ts`, the styles can affect the whole app.

```ts
createApp(App).mount("#app");
```

This means:

- Create a Vue app using `App.vue`.
- Put that Vue app inside the HTML element with `id="app"`.

In normal words: this line turns your Vue code into something visible on the webpage.

## 25. `todo.ts` Explained

Your code:

```ts
export type Todo = {
  id: number;
  title: string;
  completed: boolean;
};
```

Explanation:

```ts
export type Todo
```

This creates a TypeScript type named `Todo` and allows other files to import it.

```ts
id: number;
```

Every todo needs an `id`, and that id must be a number.

```ts
title: string;
```

Every todo needs a `title`, and that title must be text.

```ts
completed: boolean;
```

Every todo needs a `completed` value. A boolean can only be `true` or `false`.

So every todo in your app should look like this:

```ts
{
  id: 123,
  title: "Study Vue",
  completed: false
}
```

## 26. `App.vue` Script Explained

Your code starts with:

```vue
<script setup lang="ts">
```

This tells Vue:

- This is the script section.
- Use the simpler `setup` syntax.
- Use TypeScript because of `lang="ts"`.

Then you import things:

```ts
import { ref, computed } from 'vue';
```

`ref` is used for data that can change.

`computed` is used for data that is calculated from other data.

```ts
import TodoForm from './components/TodoForm.vue';
import TodoList from './components/TodoList.vue';
```

These lines import your child components so you can use them in the template.

```ts
import type { Todo } from './types/todo';
```

This imports your `Todo` type. It helps TypeScript understand what a todo should look like.

### Your Todos State

```ts
const todos = ref<Todo[]>([]);
```

This creates your todo list.

Breakdown:

- `const todos` creates a variable named `todos`.
- `ref(...)` makes it reactive.
- `<Todo[]>` means this is an array of `Todo` objects.
- `[]` means it starts empty.

In beginner words: `todos` is the list where all your todo items are stored.

### Your Filter State

```ts
const filter = ref<'all' | 'active' | 'completed'>('all');
```

This stores which filter is currently selected.

Breakdown:

- `filter` is reactive.
- It can only be `'all'`, `'active'`, or `'completed'`.
- It starts as `'all'`.

This is TypeScript helping you. If you accidentally type:

```ts
filter.value = 'finished'
```

TypeScript will complain because `'finished'` is not allowed.

### Your Add Function

```ts
const addTodo = (title: string) => {
  const newTodo: Todo = {
    id: Date.now(),
    title,
    completed: false,
  }
  todos.value.push(newTodo);
}
```

This function adds a new todo.

```ts
const addTodo = (title: string) => {
```

This creates a function named `addTodo`. It receives `title`, and TypeScript says `title` must be a string.

```ts
const newTodo: Todo = {
```

This creates a new object and says it must follow the `Todo` type.

```ts
id: Date.now(),
```

This creates a unique-ish number using the current time.

```ts
title,
```

This is shorthand for:

```ts
title: title
```

It means the todo title will use the value passed into the function.

```ts
completed: false,
```

New todos start as not completed.

```ts
todos.value.push(newTodo);
```

This adds the new todo into the reactive todo array.

Because `todos` is a `ref`, you use `todos.value` in the script.

### Your Delete Function

```ts
const deleteTodo = (id: number) =>  {
  todos.value = todos.value.filter(todo => todo.id !== id);
}
```

This removes a todo.

```ts
const deleteTodo = (id: number) => {
```

This creates a function that receives the id of the todo you want to delete.

```ts
todos.value.filter(todo => todo.id !== id)
```

This loops through all todos and keeps only the todos whose id does not match the deleted id.

Example:

```ts
todo.id !== id
```

means:

```text
Keep this todo if this todo's id is different from the id we want to delete.
```

### Your Toggle Function

```ts
const toggleTodo = (id: number) => {
  const todo = todos.value.find(todo => todo.id === id);
  if (todo) {
    todo.completed = !todo.completed;
  }
}
```

This changes a todo from active to completed, or completed back to active.

```ts
const todo = todos.value.find(todo => todo.id === id);
```

This searches for the todo with the matching id.

```ts
if (todo) {
```

This checks if the todo was found.

```ts
todo.completed = !todo.completed;
```

The `!` means opposite.

So:

- `!true` becomes `false`
- `!false` becomes `true`

That is how your checkbox toggles the todo.

### Your Update Function

```ts
const updateTodo = (payload: { id: number, title: string}) => {
  const todo = todos.value.find(todo => todo.id === payload.id);
  if (todo) {
    todo.title = payload.title;
  }
}
```

This edits the title of an existing todo.

```ts
payload: { id: number, title: string}
```

This means the function expects one object with:

- `id`
- `title`

Example payload:

```ts
{
  id: 123,
  title: "New todo title"
}
```

```ts
const todo = todos.value.find(todo => todo.id === payload.id);
```

This finds the todo that should be edited.

```ts
todo.title = payload.title;
```

This replaces the old title with the new title.

### Your Computed Filter

```ts
const filteredTodos = computed(() => {
  if (filter.value === 'active') {
    return todos.value.filter(todo => !todo.completed)
  }
  if (filter.value === 'completed') {
    return todos.value.filter(todo => todo.completed)
  }
  return todos.value;
})
```

This decides which todos should appear on the screen.

```ts
computed(() => {
```

This creates a calculated value.

```ts
if (filter.value === 'active') {
```

If the selected filter is active:

```ts
return todos.value.filter(todo => !todo.completed)
```

Return only todos that are not completed.

```ts
if (filter.value === 'completed') {
```

If the selected filter is completed:

```ts
return todos.value.filter(todo => todo.completed)
```

Return only todos that are completed.

```ts
return todos.value;
```

If neither condition matched, return all todos.

That means the `'all'` filter shows everything.

## 27. `App.vue` Template Explained

Your template:

```vue
<main class="app">
  <h1> Vue TypeScript Todo App</h1>
  <TodoForm @add-todo="addTodo" />
  <div class="filters">
    <button @click="filter = 'all'">ALL</button>
    <button @click="filter = 'active'">Active</button>
    <button @click="filter = 'completed'">Completed</button>
  </div>
  <TodoList
    :todos="filteredTodos"
    @delete-todo="deleteTodo"
    @toggle-todo="toggleTodo"
    @update-todo="updateTodo"
  />
</main>
```

Explanation:

```vue
<main class="app">
```

This is the main container. The class `app` connects to CSS in `style.css`.

```vue
<TodoForm @add-todo="addTodo" />
```

This shows your form component.

When `TodoForm` sends the `add-todo` event, `App.vue` runs `addTodo`.

```vue
<button @click="filter = 'all'">ALL</button>
```

When clicked, this changes the filter to show all todos.

```vue
<button @click="filter = 'active'">Active</button>
```

When clicked, this changes the filter to show active todos.

```vue
<button @click="filter = 'completed'">Completed</button>
```

When clicked, this changes the filter to show completed todos.

```vue
:todos="filteredTodos"
```

This passes the filtered todo list into `TodoList`.

```vue
@delete-todo="deleteTodo"
@toggle-todo="toggleTodo"
@update-todo="updateTodo"
```

These listen for events from `TodoList`.

When the child component says something happened, `App.vue` runs the correct function.

## 28. `TodoForm.vue` Explained

Your script:

```ts
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
```

Explanation:

```ts
const title = ref('')
```

This stores what the user types in the input.

```ts
const emit = defineEmits<{
    addTodo: [title: string]
}>()
```

This says the component can send an event called `addTodo`.

That event must send a string.

```ts
const trimmedTitle = title.value.trim()
```

This removes spaces from the beginning and end.

Example:

```text
"   Study Vue   "
```

becomes:

```text
"Study Vue"
```

```ts
if (!trimmedTitle) return
```

This stops empty todos from being added.

```ts
emit('addTodo', trimmedTitle)
```

This sends the new todo title to the parent component, `App.vue`.

```ts
title.value = ''
```

This clears the input after adding the todo.

Your template:

```vue
<form @submit.prevent="handleSubmit">
    <input v-model="title" type="text" placeholder="Add a todo.."/>
    <button type="submit">Add</button>
</form>
```

Explanation:

```vue
@submit.prevent="handleSubmit"
```

When the form submits, run `handleSubmit`.

`.prevent` stops the page from reloading.

```vue
v-model="title"
```

This connects the input to your `title` variable.

When you type, `title` changes automatically.

## 29. `TodoList.vue` Explained

Your code:

```ts
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
```

Explanation:

```ts
import TodoItem from './TodoItem.vue';
```

This imports the component used to display one todo.

```ts
import type { Todo } from '@/types/todo';
```

This imports your `Todo` type.

`@` is a shortcut for the `src` folder.

```ts
defineProps<{
    todos: Todo[]
}>()
```

This means `TodoList` receives a prop named `todos`.

That prop must be an array of `Todo` objects.

```ts
const emit = defineEmits<{
```

This defines events that `TodoList` can send upward.

Your template:

```vue
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
```

Explanation:

```vue
<p v-if="todos.length === 0" class="empty">No todos found.</p>
```

If the todo array is empty, show this message.

```vue
v-for="todo in todos"
```

Create one `TodoItem` for each todo.

```vue
:key="todo.id"
```

Give Vue a unique key for each item.

```vue
:todo="todo"
```

Pass the current todo into `TodoItem`.

```vue
@delete-todo="emit('deleteTodo', $event)"
```

When `TodoItem` says delete, `TodoList` passes that message upward.

`TodoList` is like a middle person between `TodoItem` and `App.vue`.

## 30. `TodoItem.vue` Explained

Your script:

```ts
const props = defineProps<{
    todo: Todo
}>()
```

This means `TodoItem` receives one todo from its parent.

```ts
const emit = defineEmits<{
    deleteTodo: [id: number]
    toggleTodo: [id: number]
    updateTodo: [payload:  { id: number, title: string}]
}>()
```

This means `TodoItem` can send three events upward.

```ts
const isEditing = ref(false)
```

This remembers if the todo is currently being edited.

At first, it is `false`, so the normal view shows.

```ts
const editedTitle = ref(props.todo.title)
```

This stores the text while editing.

It starts with the current todo title.

### Editing Functions

```ts
const startEditing = () => {
    isEditing.value = true
    editedTitle.value = props.todo.title
}
```

This turns on edit mode and fills the input with the current title.

```ts
const cancelEditing = () => {
    isEditing.value = false
    editedTitle.value = props.todo.title
}
```

This exits edit mode and restores the old title.

```ts
const saveEdit = () => {
    const trimmedTitle = editedTitle.value.trim()

    if (!trimmedTitle) return

    emit('updateTodo', {
        id: props.todo.id,
        title: trimmedTitle
    })

    isEditing.value = false
}
```

This saves the edited title.

It sends this object upward:

```ts
{
  id: props.todo.id,
  title: trimmedTitle
}
```

`App.vue` receives that object and updates the real todo.

### Template: Edit Mode

```vue
<template v-if="isEditing">
```

If `isEditing` is true, show edit mode.

```vue
<input v-model="editedTitle"
@keyup.enter="saveEdit"
@key.esc="cancelEditing"/>
```

This input is connected to `editedTitle`.

Pressing Enter saves the edit.

Learning note: `@key.esc` should usually be `@keyup.esc` in Vue.

```vue
<button @click="saveEdit">Save</button>
<button @click="cancelEditing">Cancel</button>
```

These buttons save or cancel editing.

### Template: Normal Mode

```vue
<template v-else>
```

If `isEditing` is false, show normal mode.

```vue
<input type="checkbox" :checked="todo.completed"
@change="emit('toggleTodo', todo.id)"/>
```

This checkbox shows whether the todo is completed.

When changed, it sends `toggleTodo` upward.

```vue
<span :class="{completed: todo.completed}">
    {{ todo.title }}
</span>
```

This shows the todo title.

If `todo.completed` is true, it adds the `completed` class.

```vue
<button @click="startEditing">
    Edit
</button>
```

This turns on edit mode.

```vue
<button @click="emit('deleteTodo', todo.id)">
    Delete
</button>
```

This sends a delete event upward with the todo id.

## 31. How Your Components Talk to Each Other

This is the most important part of your app.

Your data lives here:

```text
App.vue
```

Your child components do not own the main todo array.

They only ask `App.vue` to change it.

Example when adding a todo:

```text
TodoForm.vue
  emits addTodo

App.vue
  receives addTodo
  runs addTodo()
  updates todos
```

Example when deleting a todo:

```text
TodoItem.vue
  emits deleteTodo

TodoList.vue
  forwards deleteTodo

App.vue
  receives deleteTodo
  runs deleteTodo()
  updates todos
```

In simple words:

```text
Props go down.
Events go up.
```

Props send data from parent to child.

Events send messages from child to parent.

That pattern is the heart of this Todo app.
