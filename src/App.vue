<template>
  <div id="app">
    <div class="page-container">
      <div class="md-layout md-gutter">
        <div class="md-layout-item">
          <tara-task />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import TaraTask from "./components/TaraTask.vue";

export default {
  name: "app",
  components: {
    TaraTask
  }
};
</script>

<style>
#app {
}
</style>
