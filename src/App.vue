<script setup>
import { ref } from 'vue'
import WhiteboardView from './components/WhiteboardView.vue'

const boardName = ref('Draft Paper')
const performanceMode = ref('balanced')
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <div class="board-id-wrap">
        <label class="board-label" for="board-name">草稿名</label>
        <input id="board-name" v-model="boardName" class="board-name-input" type="text" maxlength="32" />
      </div>

      <div class="topbar-actions">
        <label class="mode-select">
          <span>性能模式</span>
          <select v-model="performanceMode">
            <option value="balanced">平衡</option>
            <option value="high">高性能</option>
          </select>
        </label>
      </div>
    </header>

    <WhiteboardView :board-title="boardName" :performance-mode="performanceMode" />
  </div>
</template>

<style scoped>
.app-shell {
  min-height: 100vh;
  background: #f3f4f6;
  color: #111827;
}

.topbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  padding: 12px 18px;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(15, 23, 42, 0.08);
  position: sticky;
  top: 0;
  z-index: 10;
}

.board-id-wrap,
.topbar-actions {
  display: flex;
  align-items: center;
  gap: 10px;
}

.board-label {
  font-size: 12px;
  color: #475569;
}

.board-name-input,
.mode-select select,
.share-btn {
  border: 1px solid rgba(148, 163, 184, 0.8);
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.9);
  padding: 8px 12px;
  font-size: 14px;
}

.board-name-input {
  min-width: 220px;
}

.mode-select {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
  color: #475569;
}

.share-btn {
  cursor: pointer;
  font-weight: 600;
  color: #0f172a;
}

@media (max-width: 700px) {
  .topbar {
    flex-direction: column;
    align-items: stretch;
  }

  .board-id-wrap,
  .topbar-actions {
    justify-content: space-between;
  }

  .board-name-input {
    flex: 1;
    min-width: 0;
  }
}
</style>
