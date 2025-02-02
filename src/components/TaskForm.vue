<template>
  <div class="task-form">
    <div class="d-flex justify-content-center gap-2 mb-3">
      <button class="btn btn-primary" @click="$emit('toggle-form')">
        {{ mostrarFormulario ? "Ocultar formulario" : "Nueva tarea" }}
      </button>
    </div>

    <transition name="slide">
      <div v-if="mostrarFormulario" class="mb-4 d-flex justify-content-center">
        <div style="max-width: 600px; width: 100%;">
          <div class="input-group mb-2">
            <input type="text" class="form-control" placeholder="Nombre de la tarea" v-model="localTarea.texto" />
          </div>
          <div class="input-group mb-2">
            <input type="date" class="form-control" v-model="localTarea.fecha" />
          </div>
          <div class="input-group mb-2">
            <input type="time" class="form-control" v-model="localTarea.hora" />
          </div>
          <div class="input-group mb-3">
            <select class="form-select" v-model="localTarea.prioridad">
              <option value="" disabled>Selecciona una prioridad</option>
              <option value="Alta">Alta</option>
              <option value="Media">Media</option>
              <option value="Baja">Baja</option>
            </select>
          </div>
          <button class="btn btn-success w-100" @click="handleAddTask">Agregar Tarea</button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  name: "TaskForm",
  props: {
    mostrarFormulario: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      localTarea: {
        texto: "",
        fecha: "",
        hora: "",
        prioridad: ""
      }
    }
  },
  methods: {
    handleAddTask() {
      if (
        this.localTarea.texto.trim() !== "" &&
        this.localTarea.fecha &&
        this.localTarea.hora &&
        this.localTarea.prioridad
      ) {
        this.$emit('add-task', { ...this.localTarea });
        this.localTarea = {
          texto: "",
          fecha: "",
          hora: "",
          prioridad: ""
        };
      }
    }
  }
};
</script>

<style scoped>
.slide-enter-active,
.slide-leave-active {
  transition: all 0.5s ease;
}

.slide-enter-from,
.slide-leave-to {
  max-height: 0;
  opacity: 0;
}
</style>
