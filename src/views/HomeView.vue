<template>
  <div>
    <Navbar :currentUser="currentUser" :modoOscuro="modoOscuro" @logout="logout" @toggle-dark-mode="toggleDarkMode" />

    <div class="container mt-5">
      <h1 class="text-center mb-4">Lista de tareas pendientes</h1>

      <!-- Filtro de prioridades -->
      <div :class="{ hidden: mostrarFormulario }" class="mb-4 d-flex justify-content-center">
        <div style="max-width: 600px; width: 100%;">
          <multiselect v-model="filtroPrioridades" :options="opcionesPrioridad" :multiple="true"
            placeholder="Filtrar por prioridad" />
        </div>
      </div>

      <TaskForm :mostrarFormulario="mostrarFormulario" @toggle-form="toggleFormulario" @add-task="agregarTarea" />

      <!-- Columnas de tareas -->
      <div class="row">
        <TaskColumn v-for="columna in columnas" :key="columna" :columna="columna"
          :tasks="tareasFiltradasPorColumna[columna]" :tareaArrastrada="tareaArrastrada" @drop-task="moverTarea"
          @delete-task="eliminarTarea" @toggle-details="alternarDetalles" @start-editing="activarEdicion"
          @save-task="guardarCambios" @cancel-edit="cancelarCambios" @drag-start="arrastrarTarea"
          @drag-end="finalizarArrastre" />
      </div>
    </div>

    <Footer :modoOscuro="modoOscuro" />
  </div>
</template>

<script>
import Navbar from "@/components/Navbar.vue";
import Footer from "@/components/Footer.vue";
import TaskForm from "@/components/TaskForm.vue";
import TaskColumn from "@/components/TaskColumn.vue";
import Multiselect from "vue-multiselect";
import "vue-multiselect/dist/vue-multiselect.min.css";

export default {
  name: "HomeView",
  components: {
    Navbar,
    Footer,
    TaskForm,
    TaskColumn,
    Multiselect
  },
  data() {
    return {
      currentUser: localStorage.getItem('currentUser') || null,
      tareas: JSON.parse(localStorage.getItem("tareas_" + (localStorage.getItem('currentUser') || '')))?.map((tarea) => ({
        ...tarea,
        editando: false,
      })) || [],
      filtroPrioridades: [],
      opcionesPrioridad: ["Alta", "Media", "Baja"],
      mostrarFormulario: false,
      tareaArrastrada: null,
      modoOscuro: JSON.parse(localStorage.getItem('modoOscuro')) || false,
      columnas: ["Por hacer", "En proceso", "Completado"]
    };
  },
  created() {
    if (this.modoOscuro) {
      document.body.classList.add('bg-dark', 'text-white');
    }
    if (this.currentUser) {
      const userTasks = localStorage.getItem(`tareas_${this.currentUser}`);
      if (userTasks) {
        this.tareas = JSON.parse(userTasks).map(tarea => ({
          ...tarea,
          editando: false
        }));
      }
    } else {
      this.$router.push('/login');
    }
  },
  computed: {
    tareasPorColumna() {
      return {
        "Por hacer": this.tareas.filter((tarea) => tarea.columna === "Por hacer"),
        "En proceso": this.tareas.filter((tarea) => tarea.columna === "En proceso"),
        "Completado": this.tareas.filter((tarea) => tarea.columna === "Completado"),
      };
    },
    tareasFiltradasPorColumna() {
      if (this.filtroPrioridades.length === 0) {
        return this.tareasPorColumna;
      }
      return {
        "Por hacer": this.tareasPorColumna["Por hacer"].filter((tarea) =>
          this.filtroPrioridades.includes(tarea.prioridad)
        ),
        "En proceso": this.tareasPorColumna["En proceso"].filter((tarea) =>
          this.filtroPrioridades.includes(tarea.prioridad)
        ),
        "Completado": this.tareasPorColumna["Completado"].filter((tarea) =>
          this.filtroPrioridades.includes(tarea.prioridad)
        ),
      };
    },
  },
  methods: {
    toggleFormulario() {
      this.mostrarFormulario = !this.mostrarFormulario;
    },
    toggleDarkMode() {
      this.modoOscuro = !this.modoOscuro;
      localStorage.setItem('modoOscuro', this.modoOscuro);
      if (this.modoOscuro) {
        document.body.classList.add('bg-dark', 'text-white');
      } else {
        document.body.classList.remove('bg-dark', 'text-white');
      }
    },
    agregarTarea(nuevaTarea) {
      const { texto, fecha, hora, prioridad } = nuevaTarea;
      if (texto.trim() !== "" && fecha && hora && prioridad) {
        const fechaCreacion = new Date().toLocaleString();
        this.tareas.push({
          id: Date.now(),
          texto,
          fecha,
          hora,
          prioridad,
          fechaCreacion,
          mostrarDetalles: false,
          editando: false,
          columna: "Por hacer",
        });
        this.mostrarFormulario = false;
        this.guardarTareas();
      }
    },
    alternarDetalles(tarea) {
      const index = this.tareas.findIndex(t => t.id === tarea.id);
      if (index !== -1) {
        this.tareas[index].mostrarDetalles = !this.tareas[index].mostrarDetalles;
        this.guardarTareas();
      }
    },
    activarEdicion(tarea) {
      const index = this.tareas.findIndex(t => t.id === tarea.id);
      if (index !== -1) {
        this.tareas[index].editando = true;
      }
    },
    guardarCambios(tareaEditado) {
      const index = this.tareas.findIndex(t => t.id === tareaEditado.id);
      if (index !== -1) {
        this.tareas.splice(index, 1, {
          ...tareaEditado,
          editando: false
        });
        this.guardarTareas();
      }
    },
    cancelarCambios(tarea) {
      const index = this.tareas.findIndex(t => t.id === tarea.id);
      if (index !== -1) {
        this.tareas[index].editando = false;
      }
    },
    eliminarTarea(id) {
      if (confirm("¿Estás seguro de eliminar esta tarea?") ) {
        this.tareas = this.tareas.filter((tarea) => tarea.id !== id);
        this.guardarTareas();
      }

    },
    arrastrarTarea(tarea) {
      this.tareaArrastrada = tarea;
    },
    finalizarArrastre() {
      this.tareaArrastrada = null;
    },
    moverTarea(nuevaColumna) {
      if (this.tareaArrastrada) {
        const index = this.tareas.findIndex(t => t.id === this.tareaArrastrada.id);
        if (index !== -1) {
          this.tareas[index].columna = nuevaColumna;
        }
        this.finalizarArrastre();
        this.guardarTareas();
      }
    },
    guardarTareas() {
      const tareasSinEditando = this.tareas.map((tarea) => ({
        ...tarea,
        editando: false,
      }));
      localStorage.setItem(`tareas_${this.currentUser}`, JSON.stringify(tareasSinEditando));
    },
    logout() {
      localStorage.removeItem('currentUser');
      this.currentUser = null;
      this.$router.push('/login');
    }
  },
};
</script>

<style scoped>
.hidden {
  display: none !important;
}

.container {
  margin-bottom: 80px;
}
</style>
