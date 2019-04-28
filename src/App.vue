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
    <div id="footer">
      <a href="https://github.com/puttsk/tara-su-calculator">
        <b>Github</b>
      </a>
      &nbsp;
      <a href="https://github.com/puttsk/tara-su-calculator">
        <b>About</b>
      </a>
      <br />
      &copy; 2019 ThaiSC. Currently v0.1.2 Code licensed MIT.
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
  margin-bottom: 60px;
  clear: both;
}
#footer a {
  color: white;
}
#footer {
  width: 100%;
  padding: 10px;
  background-color: #448aff;
  position: fixed;
  bottom: 0;
  left: 0;
  padding: 10px 20px;
  font-size: small;
  color: white;
}
@media (max-height: 812px) {
  #app {
    margin-bottom: 0px;
    clear: both;
  }
  #footer {
    width: 100%;
    padding: 10px;
    background-color: #448aff;
    position: relative;
    bottom: 0;
    left: 0;
    padding: 10px 20px;
    font-size: small;
    color: white;
  }
}
</style>
