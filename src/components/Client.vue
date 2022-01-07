<template>
  <div class='client'>

    <h1>Client</h1>

    <label for='username'>Username:</label>
    <!--<input type='text' id='username' name='username'>-->

    <select name='username' id='username'>
      <option value=''></option>
      <option v-for='user in users' :key='user.uid' :value='user.name'>
        {{ user.name }} <!--({{ user.role }})-->
      </option>
    </select>

    <button id='connect' v-on:click='connect()'>Connect</button> <!-- jQuery? -->
    <button id='disconnect' v-on:click='disconnect()' disabled>Disconnect</button> <!-- jQuery? -->

    <span>Socket Connection: </span>
    <span id='connection_state'>OFFLINE</span>

    <br/><br/>

    <label for='message'>Message:</label>
    <input type='text' id='message' name='message' width='160' disabled>
    <button id='send' v-on:click='send()' disabled>Send</button>

    <br/><br/>

    <div id='chat'></div>

  </div>
</template>

<script>

// socket io
import io from 'socket.io-client';

export default {
  name: 'Client',
  props: ['users'],
  data () {
    return {
      socket: null,
      myTimeout: null
    }
  },
  created: function() {

  },
  computed: {

  },
  methods: {

    connect: function() {
      var ref = this // vue

      // connect socket
      this.socket = io('http://localhost:3000')

      // username
      var username = document.querySelector('#username').value;
      this.socket.emit('add_user', username);

      // connection status
      ref.check_connection_status(this.socket);

      // chat clear
      document.querySelector('#chat').innerHTML = "";
      //this.$refs.chat.innerHTML = "";

      // events
      this.socket.on('connect', function(data) {

      });
      this.socket.on('another_client', function(data) {
        ref.chat_add(data);
      });
      this.socket.on('message', function(data) {
        console.log(data);
        ref.chat_add(data);
      });
      this.socket.on('from_server', function(data) {
        ref.chat_add(data);
      });
      this.socket.on('from_admin', function(data) {
        ref.chat_add(data);
      });
      this.socket.on('socket_disconnect', function() {
        ref.disconnect();
      });
      this.socket.on('disconnect', function() {
        document.querySelector('#connect').disabled = false;
        document.querySelector('#disconnect').disabled = true;
        document.querySelector('#username').disabled = false;
        document.querySelector('#message').disabled = true;
        document.querySelector('#send').disabled = true;
        // the lines below are here in case of a forced server shutdown
        document.querySelector('#connection_state').innerHTML = 'OFFLINE';
        clearTimeout(this.myTimeout);
      });

    },

    disconnect: function() {
      this.socket.disconnect();
    },

    send: function() {
      var ref = this // vue
      // get value from message box
      var message = document.querySelector('#message').value;
      this.socket.emit('message', message);
      // use the below code to add message to client's own chat box
      ref.chat_add('me: ' + message); // harcoded the 'me'
      // clear value from message box
      document.querySelector('#message').value = '';
    },

    chat_add: function(chat) {
      document.querySelector('#chat').innerHTML =
        document.querySelector('#chat').innerHTML + '<br/><br/>' + chat;
      //this.$refs.chat.innerHTML =
      //  this.$refs.chat.innerHTML + '<br/><br/>' + chat;
    },

    check_connection_status: function(socket) {
      var ref = this // vue
      var connection_status;
      if (socket.connected) {
        connection_status = 'ONLINE';
        // message are able to send
        document.querySelector('#connect').disabled = true;
        document.querySelector('#disconnect').disabled = false;
        document.querySelector('#username').disabled = true;
        document.querySelector('#message').disabled = false;
        document.querySelector('#send').disabled = false;
      } else {
        connection_status = 'OFFLINE';
        clearTimeout(this.myTimeout);
      }
      document.querySelector('#connection_state').innerHTML = connection_status;
      try {
        this.myTimeout = setTimeout(function() {
          ref.check_connection_status(socket);
        }, 1000); // hardcoded 1000ms
      }
      catch(err) {
        clearTimeout(this.myTimeout);
      }
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .client {
    font-family: "Lucida Console", "Courier New", monospace;
    padding: 32px;
  }
  h1 {
    /*margin: 0;*/
  }
</style>
