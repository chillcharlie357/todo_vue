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

const updateTodos = (updatedTodos) => {
  console.log('Update todos:', updatedTodos);
  todos.value = updatedTodos;
  console.log('Current todos:', todos.value);
};

</script>

<template>
  <TodoList :todos="todos"  @update:todos="updateTodos"/>
</template>
