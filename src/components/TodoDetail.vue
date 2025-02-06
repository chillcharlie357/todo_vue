<script setup>
import { marked } from 'marked';
import { computed } from 'vue';
import dayjs from 'dayjs';

const props = defineProps({
  todo: { type: Object, default: () => ({}) },
  open: { type: Boolean, default: false }
});
const emit = defineEmits(['update:open']);

// 使用 computed 实现 v-model 双向绑定
const modalOpen = computed({
  get() {
    return props.open;
  },
  set(value) {
    emit('update:open', value);
  }
});

const renderedMarkdown = (markdown) => {
  return marked(markdown || '');
};
</script>

<template>
  <a-modal v-model:open="modalOpen" :footer="null" width="90%" :title="props.todo.title">
    <p>
      <strong>到期时间:</strong>
      {{ dayjs(props.todo.dueDate).format("YYYY-MM-DD") }}
    </p>
    <p><strong>优先级:</strong> {{ props.todo.priority }}</p>
    <p v-if="props.todo.imgs && props.todo.imgs.length > 0">
      <strong>图片:</strong>
      <img
        class="image-preview-card"
        v-for="img in props.todo.imgs"
        :src="img"
        :key="img"
      />
    </p>
    <p v-if="props.todo.remark && props.todo.remark !== ''"><strong>备注:</strong></p>
    <div class="markdown-preview" v-html="renderedMarkdown(props.todo.remark)"></div>
  </a-modal>
</template>

<style scoped>
.image-preview-card {
  width: 33%;
  object-fit: cover;
  margin: 5px;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.image-preview-card:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}
.markdown-preview {
  border: 1px solid #d9d9d9;
  padding: 8px;
  min-height: 200px;
  overflow: auto;
  background-color: #fafafa;
}

</style>


