<template>
    <div>
      <md-content class="md-scrollbar" id="scrollbar">
        <md-card v-for="post in posts" :key="post.created">
          <md-card-header-text>
            <div class="md-subhead">{{sanitize(post.sender.username)}}</div>
          </md-card-header-text>
          <md-card-content>
            <span>{{sanitize(post.text)}}</span>
          </md-card-content>
        </md-card>
        <span class="md-subheading">* последние 10 сообщений.</span>
      </md-content>

      <md-field class="textarea">
        <label>Сообщение</label>
        <md-textarea v-model="message" maxlength="10500"></md-textarea>
      </md-field>
      <md-button class="md-raised md-primary" v-on:click="addPost">Добавить</md-button>
    </div>
</template>

<script>
  export default {
    name: 'PostsList',
    props: ["sendSocketMsg", 'currentRoom', 'newMsg'],
    data: function () {
      return {
        posts: [],
        message: ''
      }
    },
    watch: {
      newMsg: function(nw, old) {
        if (nw != old && nw.room == this.currentRoom) this.addToPosts(nw);
      },
      currentRoom: function(nw, old) {
        if (nw != old) this.getPosts(this.currentRoom);
      }
    },
    mounted: function() {
      this.getPosts(this.currentRoom);
    },
    updated: function() {
      let container = this.$el.querySelector("#scrollbar");
      container.scrollTop = container.scrollHeight;
    },
    methods: {
      sanitize: function (text) {
        return this.$sanitize(text);
      },
      addPost: function () {
        let msg = {
          "room": this.currentRoom,
          "text": this.sanitize(this.message),
          //"id": string // необязательный идентификатор, можно назначить на клиенте, чтобы получить подтверждение получения сообщения сервером
        };
        this.sendSocketMsg(msg);
        this.message = '';
      },
      addToPosts: function(msg) {
        this.posts.push(msg);
      },
      getPosts: async function (name) {
        try {
          let response = await fetch(`https://nane.tada.team/api/rooms/${name}/history`);
          let result = await response.json();
          let r = result["result"];
          r.sort(post => post.created);
          this.posts = r.length > 10 ? r.slice(Math.max(r.length - 10, 1)) : r;
        } catch (e) {
          this.posts = [];
          console.warn('problem with fetch');
        }
      }
    }
  }
</script>
<style scoped>
  .md-card {
    text-align: left;
    padding: 10px 10px 0 10px;
    margin: 10px 0;
  }
  .md-content {
    padding: 10px;
    max-height: 65vh;
    overflow: auto;
  }
  button {
    width: 100%;
    margin: 0;
  }
  .textarea {
    margin-bottom: 4px;
  }
</style>
