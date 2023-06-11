<template>
  <div>
    <div>
      <div class="d-flex flex-wrap">
        <span>Users connected:</span>
        <div v-if="otherNicks.length > 0" class="d-flex flex-wrap">
          <div v-for="nick of otherNicks" :key="nick" class="mx-2">
            <img
              class="img-fluid"
              width="70"
              :src="`/images/${nick}.png?${Math.random()}`"
            />
            <h5 class="text-center font-weight-bold mt-1 mb-3">{{ nick }}</h5>
          </div>
        </div>
      </div>
    </div>
    <div
      ref="container"
      class="overflow-auto p-2 shadow p-3 mb-5 bg-white rounded"
      style="height:50vh; border: 3px #C9DFF8 solid; line-height: 0.5;"
    >
      <p v-for="(m, i) of messages" :key="i">
        {{ m.nick }}: <span class="font-italic">{{ m.message }}</span>
      </p>
    </div>

    <form autocomplete="off" @submit.prevent="">
      <div class="form-group mt-2 w-50">
        <label v-html="statusText" for="nick"></label>
        <input
          v-model.trim="nick"
          id="nick"
          class="form-control text-primary"
          type="text"
          :disabled="status == 'nickname accepted'"
          @keyup.enter="sendNick"
        />
      </div>
      <div v-if="status == 'nickname accepted'" class="form-group mt-2">
        <label class="btn btn-primary mb-2"
          >Upload image
          <input type="file" @change="onFileChanged" hidden />
        </label>
        <label for="msg">Enter message below:</label>
        <input
          ref="msg"
          v-model="msg"
          id="msg"
          class="form-control"
          type="text"
          @keyup.enter="send"
        />
      </div>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  props: {
    serverUrl: {
      type: String,
    },
  },
  data() {
    return {
      ws: null,
      messages: [],
      msg: 'A message',
      nick: '',
      otherNicks: [],
      status: 'no nickname',
    };
  },
  created() {
    this.ws = new WebSocket(this.serverUrl);
    this.ws.onmessage = ({ data }) => {
      // restore object from JSON string
      let message = JSON.parse(data);
      switch (message.type) {
        case 'nickname status':
          // set status to 'nickname accepted'
          this.status = 'nickname accepted';
          // set focus to msg textbox
          this.setFocus('msg');
          break;
        case 'text':
          // restore object from JSON string (nick and message)

          // store object in array messages
          this.messages.push(message.data);
          // scroll to end
          this.scrollToEnd();
          break;
        case 'update nicknames':
        case 'connected':
          // store nickname in otherNicks
          this.otherNicks = message.data;
          break;
        default:
          alert(`Unknown message type: ${message.type}!`);
      }
    };
  },
  computed: {
    statusText() {
      switch (this.status) {
        case 'no nickname':
          return '<span class="text-primary">Please enter your nickname and press RETURN:</span>';
        case 'nickname accepted':
          return '<span>Your nickname:</span>';
        default:
          return '<span class="text-danger">Error: Unknown Status</span>';
      }
    },
  },
  methods: {
    send() {
      // send object from msg textbox as string to server
      this.ws.send(JSON.stringify({ type: 'text', msg: this.msg }));
    },
    sendNick(event) {
      if (event.key != 'Enter') {
        this.status = 'no nickname';
        return;
      }
      // send object from nick textbox as string to server
      this.ws.send(JSON.stringify({ type: 'nickname', nickname: this.nick }));
    },
    scrollToEnd() {
      const container = this.$refs['container'];
      this.$nextTick(() => (container.scrollTop = container.scrollHeight));
    },
    setFocus(element) {
      this.$nextTick(() => this.$refs[element].focus());
    },
    onFileChanged(event) {
      this.selectedFile = event.target.files[0];
      this.onUpload();
    },
    onUpload() {
      let formData = new FormData();
      formData.append('image', this.selectedFile);
      formData.append('nick', this.nick);
      axios.post('/pics', formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
        },
      });
    },
  },
};
</script>

<style scoped></style>
