<script setup>
import { ref } from 'vue';
import TodoList from './components/TodoList.vue';
import dayjs from 'dayjs';

const todos = ref ([
  {
    id: 1,
    title: '完成项目报告',
    dueDate: dayjs('2023-10-10'),
    priority: '高',
    remark: '需要包含所有最新数据',
  },
  {
    id: 2,
    title: '团队会议',
    dueDate: dayjs('2023-10-10'),
    priority: '中',
    remark: '讨论项目进展',
    finished: false,
  },
  {
    id: 3,
    title: '代码审查',
    dueDate: dayjs('2023-10-12'),
    priority: '低',
    remark: '检查新功能代码',
    finished: false,
  },
]);

const updateTodos = (updatedTodos) => {
  todos.value = updatedTodos;
};

</script>

<template>
  <TodoList :todos="todos"  @update:todos="updateTodos"/>
</template>
