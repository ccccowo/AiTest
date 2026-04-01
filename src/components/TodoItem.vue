<script setup lang="ts">
import { ref } from 'vue'

interface Todo {
  id: number
  title: string
  completed: boolean
}

const props = defineProps<{
  todo: Todo
}>()

const emit = defineEmits<{
  toggle: []
  remove: []
  update: [title: string]
}>()

const isEditing = ref(false)
const draftTitle = ref('')

function startEditing() {
  draftTitle.value = props.todo.title
  isEditing.value = true
}

function cancelEditing() {
  isEditing.value = false
  draftTitle.value = props.todo.title
}

function submitEdit() {
  const nextTitle = draftTitle.value.trim()

  if (!nextTitle) {
    return
  }

  emit('update', nextTitle)
  isEditing.value = false
}
</script>

<template>
  <li class="todo-item" :class="{ 'todo-item--done': todo.completed }">
    <label class="todo-check">
      <input
        class="todo-checkbox"
        type="checkbox"
        :checked="todo.completed"
        @change="emit('toggle')"
      />
      <span class="todo-checkmark" aria-hidden="true"></span>
    </label>

    <div class="todo-content">
      <template v-if="isEditing">
        <input
          v-model="draftTitle"
          class="todo-editor"
          type="text"
          maxlength="120"
          @keyup.enter="submitEdit"
          @keyup.esc="cancelEditing"
        />
      </template>
      <template v-else>
        <p class="todo-title">{{ todo.title }}</p>
      </template>
    </div>

    <div class="todo-actions">
      <template v-if="isEditing">
        <button class="action-button action-button--confirm" type="button" @click="submitEdit">
          保存
        </button>
        <button class="action-button" type="button" @click="cancelEditing">取消</button>
      </template>
      <template v-else>
        <button class="action-button" type="button" @click="startEditing">编辑</button>
        <button class="action-button action-button--danger" type="button" @click="emit('remove')">
          删除
        </button>
      </template>
    </div>
  </li>
</template>

<style scoped>
.todo-item {
  display: grid;
  grid-template-columns: auto 1fr auto;
  gap: 16px;
  align-items: center;
  padding: 16px 18px;
  border-radius: 22px;
  background: rgba(255, 255, 255, 0.72);
  border: 1px solid rgba(115, 92, 63, 0.12);
}

.todo-item--done {
  background: rgba(245, 242, 235, 0.74);
}

.todo-check {
  position: relative;
  display: grid;
  place-items: center;
}

.todo-checkbox {
  position: absolute;
  inset: 0;
  opacity: 0;
  cursor: pointer;
}

.todo-checkmark {
  position: relative;
  width: 24px;
  height: 24px;
  border-radius: 999px;
  border: 2px solid rgba(210, 110, 54, 0.5);
  background: #fffdfa;
  transition: background 180ms ease, transform 180ms ease, border-color 180ms ease;
}

.todo-checkbox:checked + .todo-checkmark {
  background: linear-gradient(135deg, #d7692e, #ee9b52);
  border-color: transparent;
  transform: scale(0.96);
}

.todo-checkbox:checked + .todo-checkmark::after {
  content: '';
  position: absolute;
  width: 6px;
  height: 11px;
  border-right: 2px solid #fff9f3;
  border-bottom: 2px solid #fff9f3;
  transform: rotate(45deg) translate(-1px, -1px);
}

.todo-content {
  min-width: 0;
}

.todo-title {
  margin: 0;
  font-size: 1rem;
  line-height: 1.45;
  word-break: break-word;
}

.todo-item--done .todo-title {
  color: var(--text-soft);
  text-decoration: line-through;
}

.todo-editor {
  width: 100%;
  border: 1px solid rgba(210, 110, 54, 0.25);
  border-radius: 14px;
  background: #fffefb;
  padding: 10px 12px;
  font: inherit;
  color: var(--text-strong);
}

.todo-editor:focus {
  outline: none;
  box-shadow: 0 0 0 4px rgba(210, 110, 54, 0.12);
}

.todo-actions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  justify-content: flex-end;
}

.action-button {
  border: 0;
  border-radius: 999px;
  padding: 9px 14px;
  font: inherit;
  font-size: 0.92rem;
  cursor: pointer;
  color: var(--text-strong);
  background: rgba(115, 92, 63, 0.08);
  transition: transform 180ms ease, opacity 180ms ease, background 180ms ease;
}

.action-button:hover {
  transform: translateY(-1px);
}

.action-button--confirm {
  color: #fff9f3;
  background: linear-gradient(135deg, #d7692e, #ee9b52);
}

.action-button--danger {
  color: #a1431b;
  background: rgba(185, 74, 30, 0.11);
}

@media (max-width: 720px) {
  .todo-item {
    grid-template-columns: auto 1fr;
  }

  .todo-actions {
    grid-column: 1 / -1;
  }
}
</style>
