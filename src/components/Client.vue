<template>
  <div id="client">

    <h1>Client</h1>

    <label for='username'>Username:</label>
    <input type='text' id='username' name='username' :value='username'>
    <button id='connect' @click='connect'>Connect</button> <!-- jQuery? -->
    <button id='disconnect' onclick='disconnect()' disabled>Disconnect</button> <!-- jQuery? -->

    <span>Socket Connection: </span>
    <span id='connection_state'>OFFLINE</span>

    <br/><br/>

    <label for='message'>Message:</label>
    <input type='text' id='message' name='message' width='160' disabled :value='message'>
    <button id='send' onclick='send()' disabled>Send</button>

    <br/><br/>

    <div id='chat'>{{chat}}</div>

  </div>
</template>

<script>
export default {
  name: 'Client',
  props: ['users'],
  data: function () {
    return {
      username: '',
      chat: '',
      message: '',
      socket: null,
      myTimeout: null
    }
  },
  computed: {

  },
  methods: {

    connect: function() {
      // use the vue way when more time instead of querySelector
      document.querySelector('#connect').disabled = true;
      document.querySelector('#disconnect').disabled = false;
      document.querySelector('#username').disabled = true;

      console.log('socket connection: starting...')
      this.socket = new WebSocket('ws://localhost:3000')
      //this.socket = new WebSocket(':3000')

      // username
      var username = document.querySelector('#username').value;
      socket.emit('add_user', username);

      // connection status
      check_connection_status(socket);

      // chat clear
      document.querySelector('#chat').innerHTML = "";

      // events
      this.socket.onopen = function(event) {
        console.log(event)
        console.log('websocket server: connected')
      }

      this.socket.onmessage = function(event) {
        chat_add(event.data);
      }

      this.socket.onsocket_disconnect = function(event) {
        disconnect();
      }

      this.socket.ondisconnect = function(event) {
        document.querySelector('#connect').disabled = false;
        document.querySelector('#disconnect').disabled = true;
        document.querySelector('#username').disabled = false;
        document.querySelector('#message').disabled = true;
        document.querySelector('#send').disabled = true;
        // the lines below are here in case of a forced server shutdown
        document.querySelector('#connection_state').innerHTML = 'OFFLINE';
        clearTimeout(myTimeout);
      }

    },

    check_connection_status: function(socket) {
      var connection_status;
      if (socket.connected) {
        connection_status = 'ONLINE';
        // message are able to send
        document.querySelector('#message').disabled = false;
        document.querySelector('#send').disabled = false;
      } else {
        connection_status = 'OFFLINE';
        clearTimeout(myTimeout);
      }
      document.querySelector('#connection_state').innerHTML = connection_status;
      try {
        myTimeout = setTimeout(function() {check_connection_status(socket);}, 1000); // hardcoded 1000ms
      }
      catch(err) {
        clearTimeout(myTimeout);
      }
    },

    disconnect: function() {
      socket.disconnect();
    },

    send: function() {
      // get value from message box
      var message = document.querySelector('#message').value;
      socket.emit('message', message);
      // use the below code to add message to client's own chat box
      chat_add('me: ' + message); // harcoded the 'me'
      // clear value from message box
      document.querySelector('#message').value = '';
    },

    chat_add: function(chat) {
      document.querySelector('#chat').innerHTML =
      document.querySelector('#chat').innerHTML + '<br/><br/>' + chat;
    }

  },
  created: function() {

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  body, table {
    font-family: "Lucida Console", "Courier New", monospace;
  }
  body {
    padding: 32px;
  }
  h1 {
    /*margin: 0;*/
  }
</style>
