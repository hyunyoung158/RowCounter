<template>
  <div class="project-selector">
    <select :value="modelValue" @change="onProjectSelect">
      <option :value="null" disabled>프로젝트를 선택하세요</option>
      <option v-for="project in projects" :key="project.id" :value="project.id">
        {{ project.name }}
      </option>
    </select>
    <input v-model="newProjectName" placeholder="새 프로젝트 이름" @keyup.enter="onCreateProject" />
    <button @click="onCreateProject">새 프로젝트 생성</button>
  </div>
</template>

<script>
export default {
  name: 'ProjectList',
  props: {
    projects: {
      type: Array,
      required: true,
    },
    modelValue: { // v-model을 위한 prop
      type: Number,
      default: null,
    },
  },
  emits: ['update:modelValue', 'create-project'],
  data() {
    return {
      newProjectName: '',
    };
  },
  methods: {
    onProjectSelect(event) {
      this.$emit('update:modelValue', Number(event.target.value));
    },
    onCreateProject() {
      this.$emit('create-project', this.newProjectName);
      this.newProjectName = '';
    },
  },
};
</script>

<style scoped>
.project-selector { display: flex; gap: 10px; margin-bottom: 20px; align-items: center; }
.project-selector select, .project-selector input { padding: 8px; flex-grow: 1; }
.project-selector button { padding: 8px 12px; }
</style>