<template>
  <div class="card mb-2" draggable="true" @dragstart="onDragStart" @dragend="onDragEnd">
    <div class="card-body">
      <div v-if="task.editando">
        <input type="text" class="form-control my-2" v-model="localTask.texto" placeholder="Nombre de la tarea" />
        <input type="date" class="form-control my-2" v-model="localTask.fecha" />
        <input type="time" class="form-control my-2" v-model="localTask.hora" />
        <select class="form-select my-2" v-model="localTask.prioridad">
          <option value="Alta">Alta</option>
          <option value="Media">Media</option>
          <option value="Baja">Baja</option>
        </select>
        <div class="d-flex justify-content-between mt-3">
          <button class="btn btn-success btn-sm" @click="saveChanges">Guardar cambios</button>
          <button class="btn btn-secondary btn-sm" @click="cancelChanges">Cancelar cambios</button>
        </div>
      </div>

      <div v-else>
        <h5 class="card-title d-flex align-items-center">
          {{ task.texto }}
          <i class="ms-2" :class="iconoPrioridad(task.prioridad)" :title="`Prioridad: ${task.prioridad}`"></i>
        </h5>
        <p class="card-text">
          Fecha: {{ task.fecha }} <br />
          Hora: {{ task.hora }}
        </p>
        <div v-if="task.mostrarDetalles" class="alert alert-info p-2">
          Creada el: {{ task.fechaCreacion }}
        </div>

        <div class="d-none d-md-flex justify-content-between">
          <button class="btn btn-info btn-sm" @click="toggleDetails">
            <i :class="task.mostrarDetalles ? 'bi bi-dash' : 'bi bi-plus'"></i>
          </button>
          <button class="btn btn-warning btn-sm" @click="startEditing">
            Editar
          </button>
          <button class="btn btn-danger btn-sm" @click="$emit('delete-task', task.id)">
            Eliminar
          </button>
          <button class="btn btn-outline-primary btn-sm" @click="addToGoogleCalendar" title="Añadir a Google Calendar">
            <i class="bi bi-calendar-plus"></i>
          </button>
        </div>

        <div class="d-md-none">
          <button class="btn btn-primary btn-sm w-100" @click="toggleMobileMenu">
            Acciones <i class="bi bi-three-dots"></i>
          </button>

          <div v-if="showMobileMenu" class="mobile-menu">
            <button class="menu-item" @click="startEditing">
              <i class="bi bi-pencil"></i> Editar
            </button>
            <button class="menu-item" @click="$emit('delete-task', task.id)">
              <i class="bi bi-trash"></i> Eliminar
            </button>
            <button class="menu-item" @click="addToGoogleCalendar">
              <i class="bi bi-calendar-plus"></i> Agregar a Google Calendar
            </button>
            <div class="menu-separator"></div>
            <div class="menu-title">Mover a:</div>
            <button v-for="col in ['Por hacer', 'En proceso', 'Completado']" :key="col" class="menu-item"
              :disabled="task.columna === col" @click="moveToColumn(col)">
              {{ col }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "TaskCard",
  props: {
    task: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      localTask: { ...this.task },
      showMobileMenu: false
    }
  },
  methods: {
    iconoPrioridad(prioridad) {
      switch (prioridad) {
        case "Alta":
          return "bi bi-fire text-danger";
        case "Media":
          return "bi bi-lightning-fill text-warning";
        case "Baja":
          return "bi bi-cup-hot text-success";
        default:
          return "";
      }
    },
    toggleDetails() {
      this.$emit('toggle-details', this.task);
    },
    startEditing() {
      this.$emit('start-editing', this.task);
    },
    saveChanges() {
      this.$emit('save-task', this.localTask);
    },
    cancelChanges() {
      this.$emit('cancel-edit', this.task);
    },
    onDragStart() {
      this.$emit('drag-start', this.task);
    },
    onDragEnd() {
      this.$emit('drag-end');
    },
    toggleMobileMenu() {
      this.showMobileMenu = !this.showMobileMenu;
    },
    moveToColumn(column) {
      this.$emit('update-task-column', { ...this.task, columna: column });
      this.$emit('save-task', { ...this.task });
      this.showMobileMenu = false;
    },
    addToGoogleCalendar() {
      const title = this.task.texto;
      const dateStr = this.task.fecha;
      const timeStr = this.task.hora;

      if (!dateStr || !timeStr) {
        alert("La fecha o la hora no están definidas para esta tarea.");
        return;
      }

      const startDate = new Date(`${dateStr}T${timeStr}:00`);
      const endDate = new Date(startDate.getTime() + 60 * 60 * 1000);

      function formatDate(date) {
        const pad = (n) => n < 10 ? '0' + n : n;
        return date.getFullYear().toString() +
          pad(date.getMonth() + 1) +
          pad(date.getDate()) + 'T' +
          pad(date.getHours()) +
          pad(date.getMinutes()) +
          pad(date.getSeconds());
      }

      const startFormatted = formatDate(startDate);
      const endFormatted = formatDate(endDate);

      const baseUrl = 'https://calendar.google.com/calendar/render?action=TEMPLATE';
      const text = encodeURIComponent(title);
      const dates = `${startFormatted}/${endFormatted}`;
      const url = `${baseUrl}&text=${text}&dates=${dates}`;

      window.open(url, '_blank');
    }
  },
  watch: {
    task: {
      handler(newVal) {
        this.localTask = { ...newVal };
      },
      deep: true
    }
  }
};
</script>

<style scoped>
.card {
  cursor: grab;
}

.card:active {
  cursor: grabbing;
}

.mobile-menu {
  position: absolute;
  right: 10px;
  bottom: 100%;
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  width: 200px;
  padding: 8px 0;
  margin-bottom: 5px;
}

.menu-item {
  width: 100%;
  text-align: left;
  padding: 8px 16px;
  background: none;
  display: block;
}

.menu-item:hover {
  background-color: #f8f9fa;
}

.menu-item:disabled {
  color: #aaa;
}

.menu-separator {
  height: 1px;
  background-color: #ddd;
  margin: 8px 0;
}

.menu-title {
  padding: 4px 16px;
  color: #666;
  font-size: 0.9em;
}
</style>
