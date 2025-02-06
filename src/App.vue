<script setup>
import { ref, watch } from 'vue';
import TodoList from './components/TodoList.vue';
import dayjs from 'dayjs';

// 从 localStorage 加载 todos
const loadTodos = () => {
  const savedTodos = localStorage.getItem('todos');
  const parsedTodos = savedTodos ? JSON.parse(savedTodos) : [];
  parsedTodos.forEach(todo => {
    todo.dueDate = dayjs(todo.dueDate);
  });
  return parsedTodos;
};

const todos = ref(loadTodos());

// 监听 todos 的变化，并保存到 localStorage
watch(todos, (newTodos) => {
  console.log('Save todos:', newTodos);
  localStorage.setItem('todos', JSON.stringify(newTodos));
}, { deep: true });

</script>

<template>
  <TodoList v-model:todos="todos"   class="container"/>
</template>

<style>
.class {
  margin: 0 auto;
  padding: 20px;
  width: 100%;
  height: 100%;
}
</style>
