<script setup>
import { ref } from 'vue';
import { EditOutlined, DeleteOutlined , PlusOutlined} from '@ant-design/icons-vue';
import dayjs from 'dayjs';
import { v4 as uuidv4 } from 'uuid';
import {marked} from 'marked';
import { message } from 'ant-design-vue';

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

const handleBeforeUpload = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);

    reader.onload = () => {
      if (!currentTodo.value.imgs) {
        currentTodo.value.imgs = [];
      }
      currentTodo.value.imgs.push(reader.result);
      // 取消默认上传行为
      resolve(false);
    };
    reader.onerror = () => {
      message.error('Failed to read file:', file);
      reject();
    };
  });
};

// 删除图片
const removeImage = (index) => {
  if (currentTodo.value && currentTodo.value.imgs) {
    currentTodo.value.imgs.splice(index, 1);
  }
};

</script>

<template>
  <div style="background-color: #ececec; padding: 20px">
    <a-row :gutter="16" justify="center">
      <a-col class="head"  justify="center">
        <a-button type="primary" @click="handleAddTodo">添加待办事项</a-button>
      </a-col>
    </a-row>
    <a-row :gutter="16" justify="left">
      <a-col v-for="todo in props.todos" :key="todo.id"  class="card">
        <a-card :title="todo.title" hoverable >
          <div class="card-content">
            <p><strong>到期时间:</strong> {{ dayjs(todo.dueDate).format("YYYY-MM-DD") }}</p>
            <p><strong>优先级:</strong> {{ todo.priority }}</p>
            <p v-if="todo.imgs"><strong>图片:</strong>
              <img class="image-preview-card" v-for="img in todo.imgs" :src="img" :key="img"/>
            </p>
            <p v-if="todo.remark"><strong>备注:</strong> </p>
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

    <a-modal class="modal"
      v-model:open="isModalVisible"
      :title="modelTitle"
      ok-text="确认"
      cancel-text="取消"
      width="90%"
      @ok="handleOk"
      @cancel="handleCancel">
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
      <p><strong>图片:</strong>
        <div class="image-container">
          <div class="image-wrapper" v-for="(img, index) in currentTodo.imgs" :key="index">
            <img class="image-preview" :src="img" :key="img"/>
            <delete-outlined class="delete-image" @click="removeImage(index)"/>
          </div>
        </div>
        <a-upload
          :disabled="currentTodo && currentTodo.imgs &&currentTodo.imgs.length >= 3"
          :before-upload="handleBeforeUpload"
          :show-upload-list="false"
          list-type="picture-card"
          >
          <div>
            <plus-outlined />
            <div style="margin-top: 8px">Upload</div>
          </div>
        </a-upload>
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
/* 默认显示1列 */
.card {
  margin-top: 20px;
  flex: 0 0 100%;
  max-width: 100%;
}
/* 宽屏显示3列 */
@media (min-width: 1000px) {
  .card {
    flex: 0 0 33%;
    max-width: 33%;
  }
}

.head {
  flex: 0 0 100%;
  max-width: 100%;
  margin-bottom: 16px;
}

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

.image-container {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
/* 窄屏时每行1张图片 */
@media (max-width: 999px) {
  .image-wrapper {
    flex: 0 0 100%;
  }
  .image-preview {
    width: 20%;
  }
}

/* 宽屏时每行显示3张图片 */
@media (min-width: 1000px) {
  .image-wrapper {
    flex: 0 0 calc(20% - 8px);
  }
  .image-preview {
    width: 100%;
  }
}

.image-preview-card {
  width: 33%;
  object-fit: cover;
  margin: 5px; /* 添加边距 */
  transition: transform 0.3s ease, box-shadow 0.3s ease; /* 添加过渡效果 */
}

.image-preview:hover {
  transform: scale(1.1); /* 悬浮时放大 */
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2); /* 悬浮时阴影效果 */
}
.delete-image {
  width: 20px;
}

.modal {
  width: 100%;
}
</style>
