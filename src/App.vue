<template>
  <ProjectManager />
</template>

<script>
import ProjectManager from './components/ProjectManager.vue'

export default {
  name: 'App',
  components: {
    ProjectManager
  }
}
</script>

<style>
/* 전역 스타일이 필요하다면 여기에 작성할 수 있습니다. */
/* 현재는 ProjectManager.vue의 자체 스타일을 사용하도록 비워둡니다. */
</style>
