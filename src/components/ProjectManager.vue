<template> 
  <div id="project-manager">
    <h1>🧶 Row Counter </h1>

    <!-- 프로젝트 선택 또는 생성 -->
    <div class="project-selector">
      <select v-model="currentProjectId" @change="loadProject">
        <option :value="null" disabled>프로젝트를 선택하세요</option>
        <option v-for="project in projects" :key="project.id" :value="project.id">
          {{ project.name }}
        </option>
      </select>
      <input v-model="newProjectName" placeholder="새 프로젝트 이름" @keyup.enter="createProject" />
      <button @click="createProject">새 프로젝트 생성</button>
    </div>

    <!-- 현재 프로젝트 상세 정보 -->
    <div v-if="currentProject" class="project-details">
      <h2>{{ currentProject.name }}</h2>
      
      <!-- 이미지 업로드 -->
      <div class="image-section">
        <h3>도안 또는 작업 사진</h3>
        <input type="file" @change="handleImageUpload" accept="image/*" />
        <div v-if="currentProject.image" class="image-preview-wrapper">
            <img :src="currentProject.image" alt="프로젝트 이미지" class="image-preview" />
        </div>
      </div>

      <!-- 단수 카운터 -->
      <div class="counter-section">
        <h3>카운터 목록</h3>
        <div v-if="currentProject.counters && currentProject.counters.length > 0" class="counter-list">
          <div v-for="counter in currentProject.counters" :key="counter.id" class="counter">
            <span class="counter-name">{{ counter.name }}</span>
            <div class="counter-controls">
              <button @click="decrementCounter(counter.id)">-</button>
              <span class="counter-value">{{ counter.count }}</span>
              <button @click="incrementCounter(counter.id)">+</button>
              <button v-if="counter.isDeletable !== false" @click="deleteCounter(counter.id)" class="btn-small btn-danger">삭제</button>
            </div>
          </div>
        </div>
        <div class="add-counter">
          <input v-model="newCounterName" placeholder="새 카운터 이름" @keyup.enter="addCounter" />
          <button @click="addCounter">카운터 추가</button>
        </div>
      </div>

      <!-- 메모 -->
      <div class="memo-section">
        <h3>메모</h3>
        <div class="memo-list">
          <div v-if="currentProject.memos && currentProject.memos.length === 0" class="no-memo">
            기록된 메모가 없습니다.
          </div>
          <div v-for="memo in currentProject.memos" :key="memo.id" class="memo-item">
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
    </div>
    
    <div v-else class="no-project">
      <p>새 프로젝트를 생성하거나 기존 프로젝트를 선택해주세요.</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ProjectManager',
  data() {
    return {
      projects: [],
      currentProjectId: null,
      newProjectName: '',
      newMemoText: '',
      newCounterName: '',
      editingMemoId: null, // 현재 수정 중인 메모의 ID
      editingMemoText: '', // 현재 수정 중인 메모의 텍스트
    };
  },
  computed: {
    currentProject() {
      if (!this.currentProjectId) return null;
      return this.projects.find(p => p.id === this.currentProjectId);
    }
  },
  methods: {
    // 새 프로젝트 생성
    createProject() {
      if (!this.newProjectName.trim()) {
        alert('프로젝트 이름을 입력해주세요.');
        return;
      }
      const newProject = {
        id: Date.now(), // 간단한 고유 ID 생성
        name: this.newProjectName.trim(),
        image: null,
        counters: [
          { id: Date.now(), name: '단수', count: 0, isDeletable: false } // 삭제 불가능한 기본 카운터
        ],
        memos: [], // memo를 memos 배열로 변경
      };
      this.projects.push(newProject);
      this.currentProjectId = newProject.id;
      this.newProjectName = '';
      this.saveData();
    },
    // 프로젝트 선택 시 데이터 로드
    loadProject() {
      // computed 속성에서 자동으로 처리됩니다.
      // 이 함수는 select의 change 이벤트를 위해 존재합니다.
    },
    // 카운터 증가
    incrementCounter(counterId) {
      if (!this.currentProject) return;
      const counter = this.currentProject.counters.find(c => c.id === counterId);
      if (counter) {
        counter.count++;
        this.saveData();
      }
    },
    // 카운터 감소
    decrementCounter(counterId) {
      this.saveData();
      if (!this.currentProject) return;
      const counter = this.currentProject.counters.find(c => c.id === counterId);
      if (counter && counter.count > 0) {
        counter.count--;
        this.saveData();
      }
    },
    addCounter() {
      if (!this.currentProject || !this.newCounterName.trim()) return;
      const newCounter = {
        id: Date.now(),
        name: this.newCounterName.trim(),
        count: 0,
        isDeletable: true, // 사용자가 추가한 카운터는 삭제 가능
      };
      this.currentProject.counters.push(newCounter);
      this.newCounterName = '';
      this.saveData();
    },
    deleteCounter(counterId) {
      if (!this.currentProject) return;
      if (confirm('정말로 이 카운터를 삭제하시겠습니까?')) {
        const index = this.currentProject.counters.findIndex(c => c.id === counterId);
        if (index > -1) {
          this.currentProject.counters.splice(index, 1);
          this.saveData();
        }
      }
    },
    // 이미지 업로드 처리
    handleImageUpload(event) {
      if (!this.currentProject || !event.target.files[0]) return;
      const file = event.target.files[0];
      const reader = new FileReader();
      reader.onload = (e) => {
        this.currentProject.image = e.target.result; // 이미지를 Base64 문자열로 변환하여 저장
        this.saveData();
      };
      reader.readAsDataURL(file);
    },
    // 새 메모 추가
    addMemo() {
      if (!this.currentProject || !this.newMemoText.trim()) return;
      const newMemo = {
        id: Date.now(),
        text: this.newMemoText.trim(),
        timestamp: Date.now(),
      };
      this.currentProject.memos.unshift(newMemo); // 배열의 맨 앞에 추가하여 최신순으로
      this.newMemoText = '';
      this.saveData();
    },
    // 메모 삭제
    deleteMemo(memoId) {
      if (!this.currentProject) return;
      if (confirm('정말로 이 메모를 삭제하시겠습니까?')) {
        const memoIndex = this.currentProject.memos.findIndex(m => m.id === memoId);
        if (memoIndex > -1) {
          this.currentProject.memos.splice(memoIndex, 1);
          this.saveData();
        }
      }
    },
    // 메모 수정 시작
    startEditingMemo(memo) {
      this.editingMemoId = memo.id;
      this.editingMemoText = memo.text;
    },
    // 수정된 메모 저장
    saveEditedMemo() {
      if (!this.currentProject || this.editingMemoId === null) return;
      const memo = this.currentProject.memos.find(m => m.id === this.editingMemoId);
      if (memo) {
        memo.text = this.editingMemoText.trim();
        this.saveData();
      }
      this.cancelEditingMemo();
    },
    // 메모 수정 취소
    cancelEditingMemo() {
      this.editingMemoId = null;
      this.editingMemoText = '';
    },
    formatTimestamp(timestamp) {
      return new Date(timestamp).toLocaleString('ko-KR');
    },
    // 모든 데이터를 localStorage에 저장
    saveData() {
      localStorage.setItem('knittingProjects', JSON.stringify(this.projects));
    },
    // 앱 시작 시 localStorage에서 데이터 불러오기
    loadData() {
      const savedProjects = localStorage.getItem('knittingProjects');
      if (savedProjects) {
        this.projects = JSON.parse(savedProjects);
        // 데이터 구조 마이그레이션 (rowCount -> counters)
        this.projects.forEach(p => {
          if (typeof p.rowCount !== 'undefined') {
            p.counters = [{ id: Date.now(), name: '기본 단수', count: p.rowCount }];
            delete p.rowCount;
          }
          // 기존 프로젝트에 카운터가 없는 경우 기본 카운터 추가
          if (!p.counters || p.counters.length === 0) {
            p.counters = [{ id: Date.now(), name: '단수', count: 0, isDeletable: false }];
          }
        });
        // 데이터 구조 마이그레이션 (기존 memo -> 신규 memos)
        this.projects.forEach(p => {
          if (p.memo && !p.memos) {
            p.memos = [{ id: Date.now(), text: p.memo, timestamp: Date.now() }];
            delete p.memo;
          } else if (!p.memos) {
            p.memos = [];
          }
        });

        if (this.projects.length > 0) {
          // 마지막으로 작업했던 프로젝트를 자동으로 선택
          this.currentProjectId = this.projects[this.projects.length - 1].id;
        }
      }
    }
  },
  // 컴포넌트가 생성될 때 데이터를 불러옵니다.
  created() {
    this.loadData();
  }
};
</script>

<style scoped>
/* 전체적인 스타일링 */
#project-manager {
  max-width: 800px; margin: 20px auto; padding: 20px;
  font-family: sans-serif; box-shadow: 0 2px 8px rgba(0,0,0,0.1); border-radius: 8px;
}
h1, h2, h3 { text-align: center; }

/* 프로젝트 선택/생성 */
.project-selector { display: flex; gap: 10px; margin-bottom: 20px; align-items: center; }
.project-selector select, .project-selector input { padding: 8px; flex-grow: 1; }
.project-selector button { padding: 8px 12px; }

/* 프로젝트 상세 */
.project-details { border-top: 1px solid #eee; padding-top: 20px; }
.no-project { text-align: center; color: #888; padding: 40px 0; }

/* 각 섹션 스타일 */
.image-section, .counter-section, .memo-section { margin-bottom: 30px; }

/* 이미지 */
.image-preview-wrapper { margin-top: 10px; text-align: center; }
.image-preview { max-width: 100%; max-height: 400px; border-radius: 4px; border: 1px solid #ddd; }

/* 카운터 */
.counter-list { display: flex; flex-direction: column; gap: 15px; margin-bottom: 15px; }
.counter { display: flex; justify-content: space-between; align-items: center; padding: 10px; border: 1px solid #ddd; border-radius: 4px; }
.counter-name { font-weight: bold; }
.counter-controls { display: flex; align-items: center; gap: 10px; }
.counter-controls button { font-size: 1em; width: 30px; height: 30px; border-radius: 50%; }
.counter-value { font-size: 1.2em; min-width: 30px; text-align: center; }
.add-counter { display: flex; gap: 10px; }
.add-counter input { flex-grow: 1; }

/* 메모 */
.memo-list { border: 1px solid #ddd; border-radius: 4px; padding: 10px; margin-bottom: 10px; max-height: 200px; overflow-y: auto; }
.no-memo { color: #888; text-align: center; padding: 20px 0; }
.memo-item { border-bottom: 1px solid #eee; padding: 10px 5px; display: flex; flex-direction: column; }
.memo-item:last-child { border-bottom: none; }
.memo-text { margin: 0 0 5px 0; white-space: pre-wrap; /* 줄바꿈 유지 */ }
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