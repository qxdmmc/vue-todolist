<template>
  <li :class="['todo-card', { completed: item.completed, pinned: item.isTop }]">
    <div class="todo-topline">
      <button class="check-btn" @click="$emit('toggle-complete', item)" type="button">
        <span v-if="item.completed">✓</span>
      </button>

      <div class="todo-content">
        <div class="todo-header">
          <span class="todo-status">
            {{ item.completed ? "已完成" : "待完成" }}
          </span>
          <span class="todo-time">
            最后修改：{{ formatTime(item.lastModified) }}
          </span>
        </div>

        <div v-if="editId === item.id" class="edit-row">
          <input
            :value="editText"
            @input="$emit('update:editText', ($event.target as HTMLInputElement).value.trim())"
            type="text"
            aria-label="编辑待办"
            @keyup.enter="$emit('save-edit', item)"
          />
          <div class="edit-buttons">
            <button type="button" @click="$emit('save-edit', item)">保存</button>
            <button type="button" class="muted" @click="$emit('cancel-edit')">取消</button>
          </div>
        </div>

        <p v-else class="todo-text">{{ item.text }}</p>
      </div>

      <div class="card-actions">
        <button type="button" @click="$emit('edit-item', item)">
          {{ editId === item.id ? "编辑中" : "编辑" }}
        </button>
        <button type="button" class="danger" @click="$emit('delete-item', item)">
          删除
        </button>
      </div>
    </div>

    <div class="todo-foot">
      <button type="button" class="pin-btn" @click="$emit('toggle-pin', item)">
        {{ item.isTop ? "取消置顶" : "置顶" }}
      </button>
    </div>
  </li>
</template>

<script setup lang="ts">
interface TodoItem {
  id: number;
  text: string;
  completed: boolean;
  lastModified: string;
  isTop: boolean;
}

defineProps<{
  item: TodoItem;
  editId: number | null;
  editText: string;
}>();

defineEmits<{
  'toggle-complete': [item: TodoItem];
  'toggle-pin': [item: TodoItem];
  'edit-item': [item: TodoItem];
  'save-edit': [item: TodoItem];
  'cancel-edit': [];
  'delete-item': [item: TodoItem];
  'update:editText': [value: string];
}>();

const formatTime = (value: string) => {
  return new Date(value).toLocaleString("zh-CN", {
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
    hour: "2-digit",
    minute: "2-digit"
  });
};
</script>

<style scoped>
.todo-card {
  border-radius: 24px;
  background: #ffffff;
  border: 1px solid #f3e7ff;
  box-shadow: 0 20px 45px rgba(238, 215, 255, 0.18);
  padding: 18px;
  transition: transform 0.2s ease, border-color 0.2s ease;
}

.todo-card.pinned {
  border-color: #ffd5d1;
  background: #fff4f8;
}

.todo-card.completed {
  opacity: 0.6;
}

.todo-card:hover {
  transform: translateY(-1px);
}

.todo-topline {
  display: grid;
  grid-template-columns: auto 1fr auto;
  gap: 18px;
  align-items: start;
}

.check-btn {
  width: 46px;
  height: 46px;
  border-radius: 50%;
  border: 2px solid #e6d0ff;
  background: #fff2ff;
  color: #a25f9d;
  font-size: 1rem;
  cursor: pointer;
  display: grid;
  place-items: center;
}

.todo-content {
  min-width: 0;
}

.todo-header {
  display: flex;
  flex-wrap: wrap;
  gap: 14px;
  align-items: center;
  margin-bottom: 10px;
  color: #7d6f95;
  font-size: 0.9rem;
}

.todo-status {
  padding: 4px 10px;
  border-radius: 999px;
  background: #fce8ff;
  color: #6d4b88;
  font-weight: 600;
}

.todo-text {
  margin: 0;
  line-height: 1.7;
  color: #3d324d;
  font-size: 1rem;
}

.card-actions {
  display: flex;
  flex-direction: column;
  gap: 10px;
  align-items: flex-end;
}

.card-actions button {
  border: none;
  border-radius: 16px;
  padding: 10px 16px;
  cursor: pointer;
  font-size: 0.94rem;
  transition: transform 0.16s ease;
}

.card-actions button:hover {
  transform: translateY(-1px);
}

.card-actions button:not(.danger) {
  background: #f7e2ff;
  color: #6c4d92;
}

.card-actions button.danger {
  background: #ffe2eb;
  color: #ad3d63;
}

.todo-foot {
  margin-top: 14px;
  display: flex;
  justify-content: flex-end;
}

.pin-btn {
  border: none;
  border-radius: 16px;
  padding: 10px 16px;
  background: #fff5da;
  color: #8c5b1b;
  cursor: pointer;
}

.edit-row {
  display: grid;
  gap: 12px;
}

.edit-row input {
  width: 100%;
  padding: 12px 14px;
  border-radius: 16px;
  border: 1px solid #ebd7ff;
  background: #fbf2ff;
}

.edit-buttons {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.edit-buttons button {
  border: none;
  border-radius: 16px;
  padding: 10px 14px;
  cursor: pointer;
}

.edit-buttons button.muted {
  background: #f7f1ff;
  color: #7c5f97;
}

.edit-buttons button:not(.muted) {
  background: #ffd1d9;
  color: #7f3f5f;
}

@media (max-width: 760px) {
  .todo-topline {
    grid-template-columns: auto 1fr;
  }

  .card-actions {
    flex-direction: row;
    justify-content: flex-start;
    width: 100%;
  }

  .todo-foot {
    justify-content: flex-start;
  }
}
</style>