<script setup>
import { ref } from 'vue';
import { EditOutlined, DeleteOutlined } from '@ant-design/icons-vue';
import dayjs from 'dayjs';
import { v4 as uuidv4 } from 'uuid';
import {marked} from 'marked';

const props = defineProps(['todos']);
const emit = defineEmits(['update:todos']);

const isModalVisible = ref(false);
const currentTodo = ref();
const modelTitle = ref('');

const handleEditTodo = (todo) => {
  currentTodo.value = { ...todo };
  modelTitle.value = '编辑待办事项';
  isModalVisible.value = true;
};

const handleOk = () => {
  const updatedTodos = [...props.todos];
  const index = updatedTodos.findIndex(todo => todo.id === currentTodo.value.id);
  if (index !== -1) {
    // Update existing todo
    updatedTodos[index] = { ...currentTodo.value };
    console.log('Update todo:', currentTodo.value);
  } else {
    // Add new todo
    updatedTodos.push({ ...currentTodo.value });
    console.log('Add todo:', currentTodo.value);
  }
  emit('update:todos', updatedTodos);
  console.log('Updated todos:', updatedTodos);
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
  modelTitle.value = '添加待办事项';
  isModalVisible.value = true;
};

const handleDeleteTodo = (todo) => {
  console.log('Delete todo:', todo);
  const updatedTodos = props.todos.filter(t => t.id !== todo.id);
  emit('update:todos', updatedTodos);
};

const renderedMarkdown = (markdown) => {
  return marked(markdown || '');
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
        <a-card :title="todo.title" hoverable >
          <div class="card-content">
            <p><strong>到期时间:</strong> {{ dayjs(todo.dueDate).format("YYYY-MM-DD") }}</p>
            <p><strong>优先级:</strong> {{ todo.priority }}</p>
            <p><strong>备注:</strong> </p>
            <div v-html="renderedMarkdown(todo.remark)"></div>
          </div>

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
      <p><strong>备注:</strong>
        <!-- 一列md原文件，一列渲染后 -->
        <a-row :gutter="16">
          <a-col :span="12">
            <a-textarea class="markdown-edit" v-model:value="currentTodo.remark" />
          </a-col>
          <a-col :span="12">
            <div class="markdown-preview" v-html="marked(currentTodo.remark)"></div>
          </a-col>
        </a-row>
      </p>
    </a-modal>
  </div>
</template>


<style scoped>
.card-content {
  min-height: 160px;
  max-height: 160px;
  overflow: hidden;
  text-overflow: ellipsis;
}
.markdown-edit {
  border: 1px solid #d9d9d9;
  padding: 8px;
  min-height: 200px;
  overflow: auto;
}
.markdown-preview {
  border: 1px solid #d9d9d9;
  padding: 8px;
  min-height: 200px;
  overflow: auto;
  background-color: #fafafa;
}
</style>
