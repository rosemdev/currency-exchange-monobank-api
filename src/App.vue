<template>
  <div id="app">
    <CurrencyExchange />
  </div>
</template>

<script>
  import CurrencyExchange from './components/CurrencyExchange.vue'

  export default {
    name: 'App',
    components: {
      CurrencyExchange,
    }
  }
</script>
<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@200;500;800&display=swap');
</style>
<style>
@import "./assets/styles/reset.css";
</style>

<style>
  * {box-sizing: border-box;}

  body {
    margin: 0;
    min-height: 100vh;
    overflow: hidden;
    background-image: linear-gradient(242deg, #d99c9d, #8677d9 51%, #6072cc);
  }

  #app {
    font-family: 'Poppins', sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
</style>