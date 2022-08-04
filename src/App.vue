<script setup>
import {io} from 'socket.io-client';
import {onBeforeMount, ref} from "vue";

const socket = io('http://localhost:3133/');

const messages = ref([]);
const messageText = ref('');
const joined = ref(false);
const name = ref('');
const typingDisplay = ref('');

onBeforeMount(() => {
  socket.emit('findAllMessages', {}, (response) => {
    messages.value = response;
  });

  socket.on('message', (message) => {
    messages.value.push(message);
  });

  socket.on('typing', ({name, isTyping}) => {
    if (isTyping) {
      typingDisplay.value = `${name} is typing..`;
    } else {
      typingDisplay.value = '';
    }
  });

});

const join = () => {
  socket.emit('join', {name: name.value}, () => {
    joined.value = true;
  });
}

const sendMessage = () => {
  socket.emit('createMessage', {text: messageText.value}, () => {
    messageText.value = '';
  })
};

let timeout;
const emitTyping = () => {
  socket.emit('typing', {isTyping: true});
  timeout = setTimeout(() => {
    socket.emit('typing', {isTyping: false});
  }, 2000);
}

</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label class="label-j">What's your name?</label>
        <input v-model="name">
        <button type="submit">Send</button>
      </form>
    </div>
    <div class="chat-container" v-else>

      <!--message:-->
      <div class="messages-container">
        <div v-for="message in messages" class="content-main">
          <div class="content-1">[{{ message.name }}]:</div>
          <div class="content-2">{{ message.text }}</div>
        </div>
      </div>

      <div v-if="typingDisplay">{{ typingDisplay }}</div>
      <hr>

      <!--Input message:-->
      <div class="message-input">
        <form @submit.prevent="sendMessage" class="form-message-input">
          <label class="label-j">Message</label>
          <div class="">
            <input v-model="messageText" @input="emitTyping">
            <button type="submit">Send</button>
          </div>
        </form>
      </div>

    </div>
  </div>
</template>
<style>
@import "assets/base.css";

.chat {
  padding: 20px;
  height: 100vh;
}

.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.messages-container {
  flex: 1;
}

.message-input {
  margin-top: 5px;
}

.content-main {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.content-1 {
  padding-right: 10px;
}

.label-j {
  margin-right: 10px;
}

.form-message-input{
  display: flex;
  justify-content: space-between;
}
</style>
