<template>
  <div class="container">
    <div class="block1">
      <label for="id">Your ID</label><br>
      <input type="text" v-model="sender" placeholder="Your Id"><br><br>
      <label for="senderType">Your User Type : </label> <br>
      <input type="radio" id="customer" value="Customer" v-model="senderType">
      <label for="customer">Customer</label>         
      <input type="radio" id="seller" value="Seller" v-model="senderType">
      <label for="seller">Seller</label>
      <br><br>
      <label for="id">Receiver ID</label><br>
      <input type="text" v-model="receiver" placeholder="Receiver Id"><br>
      <button type="button" :disabled="isButtonDisabled" @click="openChatRoom">Open Chat Room</button>
    </div>
    <div class="block2" :class="{'is-customer': senderType === 'Customer'}" v-if="isChatRoomOpened">
      <div class="messageReceiver">
        You are talking to : {{receiver}}
      </div>
      <div class="messageList" id="msgList">
        <div v-for="(value, index) in listMessages" :key="index">
          {{value.sender}} : {{value.text}}
        </div>
      </div>
      <div class="messageBox">
        <input type="text" v-model="message" placeholder="Chat here.." class="messageInput" @keyup.enter="sendMessage">
        <button type="button" @click="sendMessage" class="sendButton">Send</button>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import firebase from 'firebase/app'
import 'firebase/auth'
import 'firebase/database'
import { rtdbPlugin as VueFire } from 'vuefire'

Vue.use(VueFire)

const FIREBASE_CONFIG = {
    apiKey: "AIzaSyCXd8QyFdxrfPDMjjt3qwkOmncIOGLpesw",
    authDomain: "alpha-chatroom.firebaseapp.com",
    databaseURL: "https://alpha-chatroom.firebaseio.com",
    projectId: "alpha-chatroom",
    storageBucket: "alpha-chatroom.appspot.com",
    messagingSenderId: "243598382255",
    appId: "1:243598382255:web:f8bfb6db6a50512e56705a",
    measurementId: "G-LZS1P1KSBH"
  };

export default {
  name: 'Chatroom',
  data: () => {
    return {
      message : null,
      sender: null,
      senderType: null,
      receiver: null,
      listMessages: {},
      isChatRoomOpened: false
    }
  },
  computed: {
    chatRoomPath(){
      if(this.senderType === 'Seller'){
        return this.sender + '-' + this.receiver
      } else {
        return this.receiver + '-' + this.sender
      }
    },
    isButtonDisabled(){
      return !this.senderType
    }
  },
  methods: {
    init() {
      firebase.initializeApp(FIREBASE_CONFIG);
    },
    openChatRoom() {
      this.isChatRoomOpened = true
      this.$rtdbBind('listMessages', firebase.database().ref('conversations/' + this.chatRoomPath)
        .child('chats'))
        .then(()=> {
          this.scrollToBottom()
        });
    },
    sendMessage() {
      let chatPayload = {
        sendDate : firebase.database.ServerValue.TIMESTAMP,
        sender : this.sender,
        senderType : this.senderType,
        text: this.message
      }
      let path = 'conversations/' + this.chatRoomPath + '/chats'
      firebase.database().ref(path).push(chatPayload)
      this.message = null
    },
    scrollToBottom() {
      var element = document.getElementById("msgList")
      element.scrollTop = element.scrollHeight
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.init()
    })
  },
  watch: {
    listMessages: {
      handler() {
        setInterval(this.scrollToBottom(),500)
      },
      deep: true
    }
  }
}
</script>

<style>
.messageReceiver{
  font-weight: bold;
  font-size: 20px;
  background-color: #E0E0E0;
  color: #333333;
  border-radius: 12px;
  margin-bottom: 12px;
}
.messageInput{
  width: 90%;
  display: inline-block;
}
.sendButton{
  display: inline-block;
}
.messageBox{
  width: 95%;
  position: absolute;
  display: flex;
  bottom: 0
}
button{
  background-color:#70a230;
  border: none;
  color: white;
  padding: 14px;
  text-align: center;
  text-decoration: none;
  font-size: 12px;
  margin: 7px 5px;
  cursor: pointer;
  border-radius: 8px;
}
.messageList{
  overflow-y: scroll;
  height: 220px;
}
.container{
  width: 100%;
  display: flex;
}
.block1{
  display: inline-block;
  background-color: #cae5a8;
  width: 20%;
  border-radius: 8px;
  margin-right: 4px;
  height: 350px;
}
.block2{
  display: inline-block;
  background-color: #9bce59;
  width: 70%;
  border-radius: 8px;
  height: 350px;
  position: relative;
}
.is-customer{
  background-color: #004c26;
  color: white;
}
input[type=text], select {
  padding: 12px 12px;
  margin: 8px 0;
  display: inline-block;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  width: 100%;
}

input[type=submit] {
  background-color: #4CAF50;
  color: white;
  padding: 14px 16px;
  margin: 8px 0;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

input[type=radio] {
  margin-left: 10px;
}

input[type=submit]:hover {
  background-color: #45a049;
}
/* width */
::-webkit-scrollbar {
  width: 10px;
}

/* Track */
::-webkit-scrollbar-track {
  background: #f1f1f1; 
}
 
/* Handle */
::-webkit-scrollbar-thumb {
  background: #888; 
}

/* Handle on hover */
::-webkit-scrollbar-thumb:hover {
  background: #555; 
}

div {
  padding: 8px;
}
</style>