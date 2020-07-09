<template>
  <div id="app">
    <Game />
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import Game from "./components/Game.vue";

export default Vue.extend({
  name: "App",
  components: {
    Game,
  },
});
</script>

<style>
html,
body,
#app {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}
</style>
