<template>
  <GameTicTac />
</template>

<script>
import GameTicTac from './components/TicTac/TicTac.vue'

export default {
  name: 'App',
  components: {
    GameTicTac
  }
}
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
