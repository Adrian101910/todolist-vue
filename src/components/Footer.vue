<template>
  <footer class="footer" :class="modoOscuro ? 'bg-dark text-white' : 'bg-light'">
    <div class="footer-content">
      <span>Hecho por Carlos y Adrián</span>
    </div>
  </footer>
</template>

<script>
export default {
  name: "Footer",
  props: {
    modoOscuro: {
      type: Boolean,
      default: false
    }
  }
};
</script>

<style scoped>
.footer {
  position: fixed;
  bottom: 0;
  width: 100%;
  height: 60px;
  z-index: 1000;
  padding: 1rem 0;
}

.footer-content {
  height: 100%;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  padding-bottom: 10px;
}
</style>
