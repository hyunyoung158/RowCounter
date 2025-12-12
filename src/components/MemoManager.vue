<template>
  <div class="memo-section">
    <h3>메모</h3>
    <div class="memo-list">
      <div v-if="!memos || memos.length === 0" class="no-memo">
        기록된 메모가 없습니다.
      </div>
      <div v-for="memo in memos" :key="memo.id" class="memo-item">
        <div v-if="editingMemoId === memo.id">
          <textarea v-model="editingMemoText" class="memo-edit-area"></textarea>
          <div class="memo-actions">
            <button @click="saveEditedMemo" class="btn-small">저장</button>
            <button @click="cancelEditingMemo" class="btn-small btn-secondary">취소</button>
          </div>
        </div>
        <div v-else>
          <p class="memo-text">{{ memo.text }}</p>
          <div class="memo-meta">
            <span class="memo-timestamp">{{ formatTimestamp(memo.timestamp) }}</span>
            <div class="memo-actions">
              <button @click="startEditingMemo(memo)" class="btn-small">수정</button>
              <button @click="deleteMemo(memo.id)" class="btn-small btn-danger">삭제</button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <input v-model="newMemoText" @keyup.enter="addMemo" placeholder="새 메모를 입력하고 Enter" />
    <button @click="addMemo">메모 추가</button>
  </div>
</template>

<script>
export default {
  name: 'MemoManager',
  props: {
    memos: {
      type: Array,
      required: true,
    },
  },
  emits: ['update:memos'],
  data() {
    return {
      newMemoText: '',
      editingMemoId: null,
      editingMemoText: '',
    };
  },
  methods: {
    formatTimestamp(timestamp) {
      return new Date(timestamp).toLocaleString('ko-KR');
    },
    addMemo() {
      if (!this.newMemoText.trim()) return;
      const newMemo = {
        id: Date.now(),
        text: this.newMemoText.trim(),
        timestamp: Date.now(),
      };
      const updatedMemos = [newMemo, ...this.memos];
      this.$emit('update:memos', updatedMemos);
      this.newMemoText = '';
    },
    deleteMemo(memoId) {
      if (confirm('정말로 이 메모를 삭제하시겠습니까?')) {
        const updatedMemos = this.memos.filter(m => m.id !== memoId);
        this.$emit('update:memos', updatedMemos);
      }
    },
    startEditingMemo(memo) {
      this.editingMemoId = memo.id;
      this.editingMemoText = memo.text;
    },
    cancelEditingMemo() {
      this.editingMemoId = null;
      this.editingMemoText = '';
    },
    saveEditedMemo() {
      if (this.editingMemoId === null) return;
      const updatedMemos = this.memos.map(m => {
        if (m.id === this.editingMemoId) {
          return { ...m, text: this.editingMemoText.trim() };
        }
        return m;
      });
      this.$emit('update:memos', updatedMemos);
      this.cancelEditingMemo();
    },
  },
};
</script>

<style scoped>
.memo-section { margin-bottom: 30px; }
.memo-list { border: 1px solid #ddd; border-radius: 4px; padding: 10px; margin-bottom: 10px; max-height: 200px; overflow-y: auto; }
.no-memo { color: #888; text-align: center; padding: 20px 0; }
.memo-item { border-bottom: 1px solid #eee; padding: 10px 5px; display: flex; flex-direction: column; }
.memo-item:last-child { border-bottom: none; }
.memo-text { margin: 0 0 5px 0; white-space: pre-wrap; }
.memo-meta { display: flex; justify-content: space-between; align-items: center; }
.memo-timestamp { font-size: 0.8em; color: #888; flex-grow: 1; }
.memo-actions { display: flex; gap: 5px; }
.memo-edit-area { width: 100%; min-height: 60px; margin-bottom: 5px; }

.memo-section input { width: 100%; padding: 8px; margin-bottom: 10px; }
.memo-section button { width: 100%; padding: 10px; }

/* 작은 버튼 스타일 */
.btn-small {
  padding: 2px 8px;
  font-size: 0.8em;
  border-radius: 4px;
  border: 1px solid #ccc;
  background-color: #f0f0f0;
}
.btn-danger { background-color: #fdd; color: #c00; }
.btn-secondary { background-color: #e0e0e0; }
</style>