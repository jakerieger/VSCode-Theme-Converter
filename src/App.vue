<template>
  <div id="app">
    <Main/>
  </div>
</template>

<script>
import Main from './components/Main.vue'

export default {
  name: 'App',
  components: {
    Main
  },

  created() {
    document.title = "VSCode Theme Converter"
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Titillium+Web:wght@200;300&display=swap');

#app {
  font-family: 'Titillium Web', sans-serif;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  word-wrap: break-word;
}

body, html {
  background: #14141b !important;
  color: #a9b1d5 !important;
}

a {
  color: #3e599f !important;
}
</style>
