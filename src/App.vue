<template>
  <!-- <Camera /> -->
  <DeviceSelection />
</template>

<script lang="ts">
import { defineComponent } from "vue";
// import Camera from "./components/Camera.vue";
import DeviceSelection from "./components/DeviceSelection.vue";

export default defineComponent({
  name: "App",
  components: {
    // Camera,
    DeviceSelection,
  },
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
