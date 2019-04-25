<template>
  <div id="app">
    <div>
      <h1>TARA SU Calculator</h1>
      <tara-task />
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
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  margin-top: 60px;
  margin-left: 60px;
}
</style>
