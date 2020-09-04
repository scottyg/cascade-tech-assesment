<template>
  <div id="app">
    <Transactions />
  </div>
</template>

<script>
import Transactions from './components/Transactions.vue'

export default {
  name: 'App',
  components: {
    Transactions
  }
}
</script>

<style>
#app{
  font-family: 'Lato', sans-serif;
}
h1, h2, h3 {
  margin: 0;
}
</style>
