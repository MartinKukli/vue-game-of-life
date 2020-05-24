<template>
  <div id="app">
    <header class="w3-red">
      <h1>Conway's Game of Life</h1>
    </header>
    <Board msg="message"/>
  </div>
</template>

<script>
import Board from "./components/Board";

export default {
  name: "App",
  components: {
    Board
  }
};
</script>

<style>
body {
  text-align: center;
}
#app {
  display: flex;
  flex-direction: column;
  align-content: space-around;
}
</style>
