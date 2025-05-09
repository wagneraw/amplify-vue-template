<script setup lang="ts">
import '@/assets/main.css';
import { onMounted, ref } from 'vue';
import type { Schema } from '../../amplify/data/resource';
import { generateClient } from 'aws-amplify/data';

const client = generateClient<Schema>();

// create a reactive reference to the array of todos
const todos = ref<Array<Schema['Todo']["type"]>>([]);

function listTodos() {
  client.models.Todo.observeQuery().subscribe({
    next: ({ items, isSynced }) => {
      todos.value = items
     },
  }); 
}

function createTodo() {
  client.models.Todo.create({
    content: window.prompt("Todo content")
  }).then(() => {
    // After creating a new todo, update the list of todos
    listTodos();
  });
}

// Function to delete a todo when clicked
async function deleteTodo(id: string) {
  try {
    await client.models.Todo.delete({ id });
    // No need to call listTodos() as the subscription will update automatically
  } catch (error) {
    console.error('Error deleting todo:', error);
  }
}
    
// fetch todos when the component is mounted
 onMounted(() => {
  listTodos();
});

</script>

<template>
  <main>
    <h1>My todos</h1>
    <button @click="createTodo">+ new</button>
    <p class="hint">Click on a todo to delete it</p>
    <ul>
      <li 
        v-for="todo in todos" 
        :key="todo.id"
        @click="deleteTodo(todo.id)"
        class="todo-item">
        {{ todo.content }}
      </li>
    </ul>
    <div>
      🥳 App successfully hosted. Try creating a new todo.
      <br />
      <a href="https://docs.amplify.aws/gen2/start/quickstart/nextjs-pages-router/">
        Review next steps of this tutorial.
      </a>
    </div>
  </main>
<style scoped>
.todo-item {
  cursor: pointer;
  transition: background-color 0.2s;
  padding: 8px;
  border-radius: 4px;
}

.todo-item:hover {
  background-color: #f5f5f5;
  text-decoration: line-through;
}

.hint {
  font-size: 0.9em;
  color: #666;
  margin-top: 0;
}
</style>
</template>
