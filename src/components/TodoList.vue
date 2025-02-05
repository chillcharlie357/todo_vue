<script setup>
import { ref } from 'vue';
import { EditOutlined, DeleteOutlined } from '@ant-design/icons-vue';
import dayjs from 'dayjs';
import { v4 as uuidv4 } from 'uuid';

const props = defineProps(['todos']);

const isModalVisible = ref(false);
const currentTodo = ref();
const modelTitle = ref('');

const handleEditTodo = (todo) => {
  currentTodo.value = { ...todo };
  modelTitle.value = '编辑待办事项';
  isModalVisible.value = true;
};

const emit = defineEmits(['update:todos']);
const handleOk = () => {
  const updatedTodos = [...props.todos];
  const index = updatedTodos.findIndex(todo => todo.id === currentTodo.value.id);
  if (index !== -1) {
    // Update existing todo
    updatedTodos[index] = { ...currentTodo.value };
  } else {
    // Add new todo
    updatedTodos.push({ ...currentTodo.value });
  }
  emit('update:todos', updatedTodos);
  isModalVisible.value = false;
};

const handleCancel = () => {
  isModalVisible.value = false;
};

const handleAddTodo = () => {
  currentTodo.value = {
    id: uuidv4(),
    title: '',
    dueDate: dayjs(),
    priority: '中',
    remark: '',
  }
  console.log('todos:', props.todos);
  console.log('Add todo:', currentTodo.value);
  modelTitle.value = '添加待办事项';
  isModalVisible.value = true;
};

const handleDeleteTodo = (todo) => {
  console.log('Delete todo:', todo);
  const updatedTodos = props.todos.filter(t => t.id !== todo.id);
  emit('update:todos', updatedTodos);
};

</script>

<template>
  <div style="background-color: #ececec; padding: 20px">
    <a-row :gutter="16" justify="center">
      <a-col :span="24" justify="center">
        <a-button type="primary" @click="handleAddTodo">添加待办事项</a-button>
      </a-col>
    </a-row>
    <a-row :gutter="16" justify="left">
      <a-col v-for="todo in props.todos" :key="todo.id" :span="8" style="margin-top: 20px">
        <a-card :title="todo.title" hoverable>
          <p><strong>到期时间:</strong> {{ dayjs(todo.dueDate).format("YYYY-MM-DD") }}</p>
          <p><strong>优先级:</strong> {{ todo.priority }}</p>
          <p><strong>备注:</strong> {{ todo.remark }}</p>
          <template #actions>
            <edit-outlined key="edit" @click="handleEditTodo(todo)" />
            <delete-outlined key="delete" @click="handleDeleteTodo(todo)"/>
            <a-checkbox key="finish" v-model:checked="todo.finished">Finished</a-checkbox>
          </template>
        </a-card>
      </a-col>
    </a-row>

    <a-modal v-model:open="isModalVisible" :title="modelTitle" @ok="handleOk" @cancel="handleCancel">
      <p><strong>标题:</strong> <a-input v-model:value="currentTodo.title" /></p>
      <p><strong>到期时间:</strong>
        <!-- <a-input v-model:value="currentTodo.dueDate" /> -->
        <a-date-picker v-model:value="currentTodo.dueDate" />
      </p>
      <p><strong>优先级:</strong>
          <!-- <a-input v-model:value="currentTodo.priority" /> -->
        <a-select v-model:value="currentTodo.priority">
          <a-select-option value="高">高</a-select-option>
          <a-select-option value="中">中</a-select-option>
          <a-select-option value="低">低</a-select-option>
        </a-select>
      </p>
      <p><strong>备注:</strong> <a-input v-model:value="currentTodo.remark" /></p>
    </a-modal>
  </div>
</template>

