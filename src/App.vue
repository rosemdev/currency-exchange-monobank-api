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

  html, body {
    min-height: 100vh;
  }

  body {
    margin: 0;
    background-image: linear-gradient(242deg, #d99c9d, #8677d9 51%, #6072cc);
  }

  #app {
    font-family: 'Poppins', sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    min-height: 100vh;
  }

  @media (max-width: 1024px) {
    body::before {
      min-height: -webkit-fill-available;
    }
  }
</style>