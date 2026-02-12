<script setup>
import { ref } from 'vue'
import NavBar from './components/NavBar.vue'
import PokemonSearch from './components/PokemonSearch.vue'
import PokemonGame from './components/PokemonGame.vue'

const currentPage = ref('search')
</script>

<template>
  <div id="app">
    <NavBar :active="currentPage" @navigate="currentPage = $event" />
    <main>
      <PokemonSearch v-if="currentPage === 'search'" />
      <PokemonGame v-if="currentPage === 'game'" />
    </main>
  </div>
</template>

<style>
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}

#app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  width: 100%;
}

main {
  flex: 1;
  width: 100%;
  overflow-x: hidden;
}
</style>
