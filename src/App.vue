<template>
  <div id="app">
    <div class="page-container">
      <md-app>
        <md-app-content>
          <div class="md-layout">
            <tara-calculator />
          </div>
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
