<template>
  <nav class="navbar navbar-expand-lg navbar-slim"
    :class="modoOscuro ? 'navbar-dark bg-dark' : 'navbar-light bg-light'">
    <div class="container">
      <span class="navbar-brand mt-2">ToDo List</span>
      <div class="d-flex gap-2">
        <button v-if="!currentUser" class="btn btn-success mt-2" @click="goToLogin">
          Login
        </button>
        <button v-else class="btn btn-danger mt-2" @click="logout">
          Logout ({{ currentUser }})
        </button>
        <button class="btn mt-2" :class="modoOscuro ? 'btn-light text-dark' : 'btn-dark text-white'"
          @click="toggleDarkMode">
          {{ modoOscuro ? "Modo Claro" : "Modo Oscuro" }}
        </button>
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  name: "Navbar",
  props: {
    currentUser: {
      type: String,
      default: null
    },
    modoOscuro: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    goToLogin() {
      this.$router.push('/login');
    },
    logout() {
      this.$emit('logout');
    },
    toggleDarkMode() {
      this.$emit('toggle-dark-mode');
    }
  }
};
</script>

<style scoped>
</style>
