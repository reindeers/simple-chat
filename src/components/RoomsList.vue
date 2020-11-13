<template>
    <md-list>
      <md-subheader>список комнат</md-subheader>
      <md-list-item v-for="room in rooms" :key="room.name" v-on:click="$emit('change-room', room.name)">
      <span class="md-list-item-text">{{sanitize(room.name)}}</span>
      </md-list-item>
    </md-list>
</template>

<script>
  export default {
    name: 'RoomsList',
    props: ["username"],
    data: function () {
      return {
        rooms: []
      }
    },
    mounted: function() {
      this.getRoomsList();
    },
    methods: {
      sanitize: function (text) {
        return this.$sanitize(text);
      },
      getRoomsList: async function () {
        let response = await fetch('https://nane.tada.team/api/rooms')
        let result = await response.json();
        this.rooms = result["result"];
        if (this.rooms.find(room => (room.name == this.username)) == undefined)
            this.rooms.push({name: this.username, last_message: ""})
      }
    }
  }
</script>
