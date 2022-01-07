<template>
  <div id="app">
    <Client
      <Client :users=users />
  </div>
</template>

<script>
import Client from './components/Client'

// socket io
//import io from "socket.io-client";
//import VueSocketIO from "vue-socket.io";

export default {
  name: 'App',
  components: {
    Client
  },
  data: function() {
    return {
      users: [
        { uid: 1, name: 'Riel', role: 'Manager' },
        { uid: 2, name: 'Thane', role: 'Admin' },
        { uid: 3, name: 'Laszlo', role: 'User' },
        { uid: 4, name: 'Alexa', role: 'User' },
        { uid: 5, name: 'Sam', role: 'User' },
      ]
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /*text-align: center;*/
  color: #2c3e50;
  /*margin-top: 60px;*/
}
</style>
