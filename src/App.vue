<script setup lang="ts">
import { computed, ref, watch } from 'vue'

import TodoItem from './components/TodoItem.vue'

interface Todo {
  id: number
  title: string
  completed: boolean
}

const STORAGE_KEY = 'vue3-todo-list'

function loadTodos(): Todo[] {
  if (typeof window === 'undefined') {
    return []
  }

  const raw = window.localStorage.getItem(STORAGE_KEY)

  if (!raw) {
    return [
      { id: 1, title: '完成第一个 Vue 3 Todo', completed: false },
      { id: 2, title: '试试编辑、删除和勾选功能', completed: true },
    ]
  }

  try {
    const parsed = JSON.parse(raw) as Todo[]

    if (!Array.isArray(parsed)) {
      return []
    }

    return parsed.filter(
      (item): item is Todo =>
        typeof item.id === 'number' &&
        typeof item.title === 'string' &&
        typeof item.completed === 'boolean',
    )
  } catch {
    return []
  }
}

const todos = ref<Todo[]>(loadTodos())
const newTodoTitle = ref('')

const totalCount = computed(() => todos.value.length)
const completedCount = computed(() => todos.value.filter((todo) => todo.completed).length)
const pendingCount = computed(() => totalCount.value - completedCount.value)

watch(
  todos,
  (value) => {
    window.localStorage.setItem(STORAGE_KEY, JSON.stringify(value))
  },
  { deep: true },
)

function addTodo() {
  const title = newTodoTitle.value.trim()

  if (!title) {
    return
  }

  todos.value.unshift({
    id: Date.now(),
    title,
    completed: false,
  })

  newTodoTitle.value = ''
}

function toggleTodo(id: number) {
  const target = todos.value.find((todo) => todo.id === id)

  if (!target) {
    return
  }

  target.completed = !target.completed
}

function updateTodo(id: number, title: string) {
  const target = todos.value.find((todo) => todo.id === id)
  const nextTitle = title.trim()

  if (!target || !nextTitle) {
    return
  }

  target.title = nextTitle
}

function removeTodo(id: number) {
  todos.value = todos.value.filter((todo) => todo.id !== id)
}

function clearCompleted() {
  todos.value = todos.value.filter((todo) => !todo.completed)
}
</script>

<template>
  <main class="app-shell">
    <section class="todo-panel">
      <header class="hero">
        <p class="eyebrow">Vue 3 Todo List</p>
        <h1>把任务清单保持在一个清晰的节奏里。</h1>
        <p class="hero-copy">
          支持新增、编辑、删除和完成打钩，刷新页面后也会保留当前记录。
        </p>
      </header>

      <form class="composer" @submit.prevent="addTodo">
        <label class="composer-label" for="new-todo">新增任务</label>
        <div class="composer-row">
          <input
            id="new-todo"
            v-model="newTodoTitle"
            class="composer-input"
            type="text"
            maxlength="120"
            placeholder="例如：整理今天的开发事项"
          />
          <button class="primary-button" type="submit">添加</button>
        </div>
      </form>

      <section class="stats">
        <article>
          <span>全部</span>
          <strong>{{ totalCount }}</strong>
        </article>
        <article>
          <span>待完成</span>
          <strong>{{ pendingCount }}</strong>
        </article>
        <article>
          <span>已完成</span>
          <strong>{{ completedCount }}</strong>
        </article>
      </section>

      <section class="list-card">
        <div class="list-header">
          <h2>任务列表</h2>
          <button
            class="ghost-button"
            type="button"
            :disabled="completedCount === 0"
            @click="clearCompleted"
          >
            清除已完成
          </button>
        </div>

        <p v-if="totalCount === 0" class="empty-state">
          现在还是空的，先添加一条任务开始。
        </p>

        <TransitionGroup v-else name="todo-list" tag="ul" class="todo-list">
          <TodoItem
            v-for="todo in todos"
            :key="todo.id"
            :todo="todo"
            @toggle="toggleTodo(todo.id)"
            @remove="removeTodo(todo.id)"
            @update="updateTodo(todo.id, $event)"
          />
        </TransitionGroup>
      </section>
    </section>
  </main>
</template>

<style scoped>
.app-shell {
  min-height: 100vh;
  display: grid;
  place-items: center;
  padding: 32px 20px;
}

.todo-panel {
  width: min(100%, 880px);
  display: grid;
  gap: 24px;
}

.hero {
  display: grid;
  gap: 10px;
}

.eyebrow {
  margin: 0;
  font-size: 0.78rem;
  font-weight: 700;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  color: var(--accent-strong);
}

.hero h1 {
  margin: 0;
  font-size: clamp(2.2rem, 5vw, 4.2rem);
  line-height: 0.95;
  letter-spacing: -0.05em;
  max-width: 10ch;
}

.hero-copy {
  margin: 0;
  max-width: 42rem;
  color: var(--text-soft);
  font-size: 1rem;
}

.composer,
.list-card,
.stats article {
  backdrop-filter: blur(18px);
  background: rgba(255, 248, 240, 0.72);
  border: 1px solid rgba(115, 92, 63, 0.12);
  box-shadow: 0 24px 60px rgba(94, 63, 32, 0.12);
}

.composer,
.list-card {
  border-radius: 28px;
  padding: 24px;
}

.composer {
  display: grid;
  gap: 14px;
}

.composer-label,
.list-header h2 {
  margin: 0;
  font-size: 0.95rem;
  font-weight: 700;
}

.composer-row {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 12px;
}

.composer-input {
  width: 100%;
  border: 1px solid rgba(115, 92, 63, 0.18);
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.84);
  padding: 15px 18px;
  font: inherit;
  color: var(--text-strong);
  transition: border-color 180ms ease, box-shadow 180ms ease, transform 180ms ease;
}

.composer-input:focus {
  outline: none;
  border-color: rgba(210, 110, 54, 0.72);
  box-shadow: 0 0 0 4px rgba(210, 110, 54, 0.12);
  transform: translateY(-1px);
}

.stats {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 14px;
}

.stats article {
  border-radius: 22px;
  padding: 18px 20px;
  display: grid;
  gap: 6px;
}

.stats span {
  color: var(--text-soft);
  font-size: 0.92rem;
}

.stats strong {
  font-size: clamp(1.8rem, 4vw, 2.4rem);
  line-height: 1;
}

.list-card {
  display: grid;
  gap: 18px;
}

.list-header {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  align-items: center;
}

.todo-list {
  list-style: none;
  display: grid;
  gap: 12px;
  padding: 0;
  margin: 0;
}

.empty-state {
  margin: 12px 0 0;
  border-radius: 20px;
  border: 1px dashed rgba(115, 92, 63, 0.24);
  padding: 28px;
  text-align: center;
  color: var(--text-soft);
  background: rgba(255, 255, 255, 0.4);
}

.primary-button,
.ghost-button {
  border: 0;
  border-radius: 999px;
  font: inherit;
  font-weight: 700;
  cursor: pointer;
  transition: transform 180ms ease, opacity 180ms ease, background 180ms ease;
}

.primary-button {
  padding: 0 22px;
  background: linear-gradient(135deg, #d7692e, #ee9b52);
  color: #fffaf3;
  min-height: 52px;
}

.ghost-button {
  padding: 10px 16px;
  color: var(--accent-strong);
  background: rgba(210, 110, 54, 0.08);
}

.primary-button:hover,
.ghost-button:hover:not(:disabled) {
  transform: translateY(-1px);
}

.ghost-button:disabled {
  cursor: not-allowed;
  opacity: 0.45;
}

.todo-list-enter-active,
.todo-list-leave-active,
.todo-list-move {
  transition: transform 220ms ease, opacity 220ms ease;
}

.todo-list-enter-from,
.todo-list-leave-to {
  opacity: 0;
  transform: translateY(12px);
}

@media (max-width: 720px) {
  .app-shell {
    padding: 20px 14px;
  }

  .composer-row,
  .stats {
    grid-template-columns: 1fr;
  }

  .primary-button {
    min-height: 48px;
  }

  .list-header {
    flex-direction: column;
    align-items: stretch;
  }
}
</style>
