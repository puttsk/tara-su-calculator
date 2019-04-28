<template>
  <div id="app">
    <div class="page-container">
      <md-app>
        <md-app-toolbar class="md-primary">
          <span class="md-title">Tara Pricing Calculator</span>
        </md-app-toolbar>
        <md-app-content>
          <tara-calculator />
        </md-app-content>
      </md-app>
    </div>
  </div>
</template>

<script>
import TaraCalculator from "./components/TaraCalculator.vue";

export default {
  name: "app",
  components: {
    TaraCalculator
  }
};
</script>

<style>
#app {
}
</style>
