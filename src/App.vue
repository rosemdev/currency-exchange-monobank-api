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
    position: relative;
  }

  body::before {
    position: absolute;
    content: '';
    left: 0;
    right: 0;
    height: 100vh;
    background-image: linear-gradient(242deg, #d99c9d, #8677d9 51%, #6072cc);
    background-repeat: no-repeat;
    z-index: -1;
  }

  #app {
    font-family: 'Poppins', sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
</style>