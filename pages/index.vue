<template>
  <div class="home-container">
    <div class="projects-area">
      <ProjectCart
        v-for="project in projects"
        :key="project.id"
        :projectData="project"
        @openEditModal="openEditModal(project)"
      />
    </div>

    <ProjectModal
      v-if="projectModal"
      v-model="projectName"
      @closeModal="closeModal"
      @updateProject="updateProject"
    />
  </div>
</template>

<script>
export default {
  name: 'IndexPage',

  layout: 'mainLayout',

  data() {
    return {
      token: "",
      projects: [],
      projectName: "",
      editingProjectId: null,
      projectModal: false
    }
  },

  mounted() {
    this.token = localStorage.getItem("token")
    if (!this.token) this.$router.push("/login")

    this.getProjects()
  },

  methods: {
    openEditModal(project) {
      this.projectName = project.name
      this.editingProjectId = project.id
      this.projectModal = true
    },

    closeModal() {
      this.projectName = ""
      this.editingProjectId = null
      this.projectModal = false
    },

    getProjects() {
      this.$axios.$get("https://api.quwi.com/v2/projects-manage/index?filters[is_active]=1&sort=dta_create",
        { headers: {"Authorization" : `Bearer ${this.token}`} }
      )
        .then(response => {
          this.projects = response.projects
        })
        .catch(err => {
          console.error(err.message)
        })
    },

    updateProject() {
      this.$axios.$put(`https://api.quwi.com/v2/projects-manage/update?id=${this.editingProjectId}`,
        { name: this.projectName },
        { headers: {"Authorization" : `Bearer ${this.token}`} }
      )
        .then(() => {
          this.projectName = ""
          this.editingProjectId = null
          this.getProjects()
          this.projectModal = false
        })
        .catch(err => {
          console.error(err.message)
        })
    }
  }
}
</script>

<style lang="scss" scoped>
.home-container {
  background-color: #f2f2f2;
  height: 90vh;
  padding-top: 10vh;
  display: flex;
  justify-content: center;

  .projects-area {
    width: 50%;
  }
}
</style>
