<template>
  <md-content class="container">
    <div class="username md-layout-item md-size-70" v-if="state == STATE_DICT.INITIAL">
      <md-field>
        <label>Введите ваше имя</label>
        <md-input v-model="username0" required maxlength="50"></md-input>
        <span class="md-error">Обязательное поле</span>
      </md-field>
      <md-button class="md-raised md-primary" v-on:click="add">Добавить</md-button>
    </div>
    <div v-if="state != STATE_DICT.INITIAL" class="posts md-layout-item md-size-70">
      <PostsList
          v-bind:sendSocketMsg="sendSocketMsg"
          v-bind:currentRoom="currentRoom"
          v-bind:newMsg = "newMessage"
          v-if="state == STATE_DICT.GET_ROOMS">
      </PostsList>
    </div>

    <md-card class="menu">
      <md-card-content>
        <RoomsList
          v-if="state && state != STATE_DICT.INITIAL"
          v-on:change-room="setCurrentRoom($event)"
          v-bind:username="username"></RoomsList>
      </md-card-content>
    </md-card>
  </md-content>
</template>

<script>
  import RoomsList from './RoomsList.vue'
  import PostsList from './PostsList.vue'

  export default {
    name: 'Chat',
    components: {
      RoomsList, PostsList
    },
    data: function () {
      return {
          sendSocketMsg: '',
          username0: '',
          username: '',
          currentRoom: '',
          STATE_DICT: {
            INITIAL: 0,
            START_CONNECTION: 1,
            GET_NEW_MSG: 2,
            GET_ROOMS: 3
          },
          state: '',
          newMessage: ''
      }
    },
    mounted: function() {
      this.state = this.STATE_DICT.INITIAL;
    },
    methods: {
      sanitize: function (text) {
        return this.$sanitize(text);
      },
      setCurrentRoom: function(name) {
        this.currentRoom = name;
        this.state = this.STATE_DICT.GET_ROOMS;
      },
      add: function () {
        this.username = this.sanitize(this.username0);
        let socket = new WebSocket(`wss://nane.tada.team/ws?username=${this.username}`);

        socket.onopen = async () => {
          console.log('Connection start');
          this.state = this.STATE_DICT.START_CONNECTION;
        };

        socket.onmessage = (e) => {
          console.log(`Get data from server: ${e.data}`);
          this.newMessage = JSON.parse(e.data);
        };

        socket.onclose = function(event) {
          if (event.wasClean) {
            console.log(`Connection end with code=${event.code} and reason=${event.reason}`);
          } else {
            console.log('Connection end');
          }
        };

        socket.onerror = function(error) {
          console.log(`[error] ${error.message}`);
        };

        this.sendSocketMsg = function(msg) {
          socket.send(JSON.stringify(msg));
        }

      }
    }
  }

</script>

<style scoped>
  .md-content {
    padding: 10px 20px;
    width: 100%;
    min-height: 100vh;
  }
  .username {
    display: inline-flex;
    align-items: baseline;
  }
  .posts {
    flex-direction: column;
  }
  .container {
    display: flex;
    justify-content: space-between;
  }
  .menu {
    min-width: 20vw;
    min-height: 90vh;
    height: 100%;
  }
</style>
