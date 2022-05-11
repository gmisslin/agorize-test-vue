<template>
  <div id="app">
    <h1>Agorize Test</h1>
    <EventAvailabilities> </EventAvailabilities>
  </div>
</template>

<script>
import EventAvailabilities from "./components/EventsAvailabilities.vue"
export default {
  name: 'App',
  components:{
    EventAvailabilities
  }  
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

#app h1 {
  text-align: center;
}
</style>
