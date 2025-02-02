<template>
  <div class="col-md-4">
    <h3 class="text-center">
      {{ columna }} <span class="badge bg-secondary">{{ tasks.length }}</span>
    </h3>
    <div class="p-3 bg-light border position-relative" @dragover.prevent="allowDrop" @drop="dropTask">
      <!-- Texto "Arrastrar aquí" -->
      <div v-if="tareaArrastrada && tareaArrastrada.columna !== columna"
        class="p-3 text-center text-muted border border-primary rounded mb-2"
        style="height: 80px; display: flex; align-items: center; justify-content: center;">
        Arrastrar aquí
      </div>

      <!-- Mensaje de columna vacía -->
      <div v-if="!tareaArrastrada && tasks.length === 0" class="text-center text-muted p-3"
        style="border: 2px dashed #ccc; border-radius: 5px;">
        No hay tareas en esta columna
      </div>

      <!-- Listamos cada tarea con el componente TaskCard -->
      <TaskCard v-for="task in tasks" :key="task.id" :task="task" @delete-task="$emit('delete-task', $event)"
        @toggle-details="$emit('toggle-details', $event)" @start-editing="$emit('start-editing', $event)"
        @save-task="$emit('save-task', $event)" @cancel-edit="$emit('cancel-edit', $event)"
        @drag-start="$emit('drag-start', $event)" @drag-end="$emit('drag-end')" />
    </div>
  </div>
</template>

<script>
import TaskCard from "./TaskCard.vue";

export default {
  name: "TaskColumn",
  components: {
    TaskCard
  },
  props: {
    columna: {
      type: String,
      required: true
    },
    tasks: {
      type: Array,
      default: () => []
    },
    tareaArrastrada: {
      type: Object,
      default: null
    }
  },
  methods: {
    allowDrop(event) {
      event.preventDefault();
    },
    dropTask() {
      this.$emit('drop-task', this.columna);
    }
  }
};
</script>

<style scoped>
/* Estilos locales si son necesarios */
</style>
