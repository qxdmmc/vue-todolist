<template>
  <div class="app-shell">
    <header class="app-header">
      <div>
        <p class="app-subtitle">轻松管理你的待办</p>
        <h1>TodoList</h1>
      </div>
      <div class="header-meta">
        <span>总待办：{{ todos.length }}</span>
        <span>已完成：{{ completedCount }}</span>
      </div>
    </header>

    <section class="panel">
      <form class="todo-form" @submit.prevent="addTodo">
        <input
          v-model.trim="newTodoText"
          type="text"
          placeholder="输入待办内容，按回车添加"
          aria-label="输入待办内容"
        />
        <button type="submit">新增</button>
      </form>

      <div class="filters-row">
        <div class="filter-group">
          <button
            type="button"
            :class="{ active: filterMode === 'all' }"
            @click="filterMode = 'all'"
          >
            全部
          </button>
          <button
            type="button"
            :class="{ active: filterMode === 'active' }"
            @click="filterMode = 'active'"
          >
            未完成
          </button>
          <button
            type="button"
            :class="{ active: filterMode === 'completed' }"
            @click="filterMode = 'completed'"
          >
            已完成
          </button>
        </div>

        <div class="filter-group">
          <select v-model="sortMode" aria-label="排序方式">
            <option value="top">置顶优先</option>
            <option value="newest">最新修改</option>
            <option value="oldest">最早修改</option>
          </select>

          <input
            v-model.trim="searchKeyword"
            type="search"
            placeholder="搜索待办"
            aria-label="搜索待办"
          />
        </div>
      </div>
    </section>

    <section class="panel">
      <div v-if="todos.length === 0" class="empty-state">
        <p>当前还没有待办，快去添加第一个任务吧！</p>
      </div>

      <div v-else-if="filteredTodos.length === 0" class="empty-state">
        <p>没有匹配到当前筛选/搜索结果。</p>
      </div>

      <ul v-else class="todo-list">
        <TodoItem
          v-for="item in filteredTodos"
          :key="item.id"
          :item="item"
          :edit-id="editId"
          :edit-text="editText"
          @toggle-complete="toggleComplete"
          @toggle-pin="togglePin"
          @edit-item="editItem"
          @save-edit="saveEdit"
          @cancel-edit="cancelEdit"
          @delete-item="deleteItem"
          @update:edit-text="editText = $event"
        />
      </ul>
    </section>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, watch } from "vue";
import TodoItem from "./components/TodoItem.vue";

interface TodoItem {
  id: number;
  text: string;
  completed: boolean;
  lastModified: string;
  isTop: boolean;
}

const STORAGE_KEY = "macaron-todo-list";
const todos = ref<TodoItem[]>([]);
const newTodoText = ref("");
const searchKeyword = ref("");
const filterMode = ref<"all" | "active" | "completed">("all");
const sortMode = ref<"top" | "newest" | "oldest">("top");
const editId = ref<number | null>(null);
const editText = ref("");

const completedCount = computed(() => todos.value.filter((item) => item.completed).length);

const loadTodos = () => {
  const raw = localStorage.getItem(STORAGE_KEY);
  if (!raw) return;
  try {
    const saved = JSON.parse(raw) as TodoItem[];
    todos.value = Array.isArray(saved) ? saved : [];
  } catch {
    todos.value = [];
  }
};

const saveTodos = () => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value));
};

onMounted(() => {
  loadTodos();
});

watch(todos, saveTodos, { deep: true });

const currentTime = () => new Date().toISOString();

const addTodo = () => {
  const content = newTodoText.value.trim();
  if (!content) return;
  todos.value.unshift({
    id: Date.now(),
    text: content,
    completed: false,
    lastModified: currentTime(),
    isTop: false
  });
  newTodoText.value = "";
};

const toggleComplete = (item: TodoItem) => {
  item.completed = !item.completed;
  item.lastModified = currentTime();
};

const togglePin = (item: TodoItem) => {
  item.isTop = !item.isTop;
  item.lastModified = currentTime();
};

const editItem = (item: TodoItem) => {
  editId.value = item.id;
  editText.value = item.text;
};

const saveEdit = (item: TodoItem) => {
  const text = editText.value.trim();
  if (!text) return;
  item.text = text;
  item.lastModified = currentTime();
  editId.value = null;
  editText.value = "";
};

const cancelEdit = () => {
  editId.value = null;
  editText.value = "";
};

const deleteItem = (item: TodoItem) => {
  const confirmed = window.confirm("确定删除此待办吗？");
  if (!confirmed) return;
  todos.value = todos.value.filter((todo) => todo.id !== item.id);
};

const filteredTodos = computed(() => {
  let list = [...todos.value];

  if (filterMode.value === "active") {
    list = list.filter((item) => !item.completed);
  } else if (filterMode.value === "completed") {
    list = list.filter((item) => item.completed);
  }

  const keyword = searchKeyword.value.trim().toLowerCase();
  if (keyword) {
    list = list.filter((item) => item.text.toLowerCase().includes(keyword));
  }

  if (sortMode.value === "top") {
    list.sort((a, b) => {
      if (a.isTop !== b.isTop) {
        return a.isTop ? -1 : 1;
      }
      return new Date(b.lastModified).getTime() - new Date(a.lastModified).getTime();
    });
  } else if (sortMode.value === "newest") {
    list.sort((a, b) => new Date(b.lastModified).getTime() - new Date(a.lastModified).getTime());
  } else {
    list.sort((a, b) => new Date(a.lastModified).getTime() - new Date(b.lastModified).getTime());
  }

  return list;
});
</script>

<style>
.app-shell {
  width: min(1080px, calc(100% - 32px));
  margin: 0 auto;
  padding: 28px 0 40px;
}

.app-header {
  display: flex;
  justify-content: space-between;
  gap: 24px;
  align-items: flex-end;
  margin-bottom: 22px;
  padding: 24px;
  border-radius: 28px;
  background: #fff7fd;
  box-shadow: 0 20px 50px rgba(183, 141, 202, 0.18);
}

.app-subtitle {
  margin: 0 0 6px;
  color: #9b7fbf;
  font-size: 0.95rem;
}

.app-header h1 {
  margin: 0;
  font-size: clamp(2rem, 2.8vw, 3rem);
  letter-spacing: -0.05em;
}

.header-meta {
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
  color: #7d6f95;
  font-size: 0.95rem;
}

.panel {
  background: #ffffffcc;
  border: 1px solid #f2e9ff;
  border-radius: 28px;
  padding: 24px;
  margin-bottom: 20px;
  box-shadow: 0 18px 50px rgba(234, 201, 238, 0.2);
}

.todo-form {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 14px;
  margin-bottom: 18px;
}

.todo-form input,
.filters-row input,
.filters-row select {
  width: 100%;
  border: 1px solid #f1e4ff;
  border-radius: 18px;
  padding: 14px 16px;
  background: #fcf8ff;
  color: #3c3a49;
  transition: border-color 0.2s ease;
}

.todo-form input:focus,
.filters-row input:focus,
.filters-row select:focus {
  outline: none;
  border-color: #c9a6ff;
}

.todo-form button {
  border: none;
  border-radius: 18px;
  padding: 14px 20px;
  background: #f8b7d6;
  color: #3c3a49;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.16s ease, box-shadow 0.16s ease;
}

.todo-form button:hover {
  transform: translateY(-1px);
  box-shadow: 0 12px 26px rgba(255, 171, 197, 0.24);
}

.filters-row {
  display: flex;
  flex-wrap: wrap;
  gap: 14px;
  justify-content: space-between;
  align-items: center;
}

.filter-group {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.filter-group button {
  border: 1px solid transparent;
  border-radius: 999px;
  background: #f7e9ff;
  color: #71579a;
  padding: 10px 18px;
  cursor: pointer;
  transition: background 0.2s ease, transform 0.2s ease;
}

.filter-group button.active {
  background: #ffd7e9;
  border-color: #f8b7d6;
  color: #5f2f50;
}

.filter-group button:hover {
  transform: translateY(-1px);
}

.filter-group select {
  max-width: 180px;
  appearance: none;
  cursor: pointer;
}

.todo-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: grid;
  gap: 16px;
}

.empty-state {
  min-height: 180px;
  border-radius: 22px;
  background: #fff8f4;
  border: 1px dashed #ffd6b6;
  display: grid;
  place-items: center;
  color: #8f6a56;
  text-align: center;
  padding: 40px 20px;
  font-size: 1rem;
}

@media (max-width: 760px) {
  .app-shell {
    width: calc(100% - 20px);
    padding: 18px 0 30px;
  }

  .app-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .todo-form {
    grid-template-columns: 1fr;
  }

  .filters-row {
    flex-direction: column;
    align-items: stretch;
  }
}
</style>