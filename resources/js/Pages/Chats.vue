<template>
  <app-layout>
    <template #header>
      <h2 class="font-semibold text-xl text-gray-800 leading-tight">Chats</h2>
    </template>

    <div class="container">
      <div class="row mt-5">
        <div class="col-md-8">
          <div class="card text-dark bg-light mb-3">
            <div class="card-header">Messages</div>
            <div class="card-body">
              <div style="height: 300px; overflow-y: auto" v-chat-scroll>
                <ul>
                  <li class="p-2" v-for="message in messages" :key="message.id">
                    <div :class="{ rightMsg: user.id === message.user.id }">
                      <strong>{{ message.user.name }}: </strong>
                      {{ message.message }}
                    </div>
                  </li>
                </ul>
              </div>
              <input
                class="form-control"
                type="text"
                placeholder="Enter your message..."
                v-model="message"
                @keyup.enter="sendMessage()"
                @keydown="sendTypingEvent()"
              />
              <span v-if="activeUser" class="text-success"
                >{{ activeUser.name }} is typing...</span
              >
            </div>
          </div>
        </div>
        <div class="col-md-4">
          <div class="card text-dark bg-light mb-3">
            <div class="card-header">Active Users</div>
            <div class="card-body">
              <ul>
                <li class="text-success" v-for="user in users" :key="user.id">
                  {{ user.name }}
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
  </app-layout>
</template>

<script>
import AppLayout from "@/Layouts/AppLayout";

export default {
  components: {
    AppLayout,
  },
  props: ["user"],
  data() {
    return {
      messages: [],
      message: "",
      users: [],
      activeUser: "",
      typingTimer: false,
    };
  },
  created() {
    this.fetchMessages();

    Echo.join("chat")
      .here((user) => {
        this.users = user;
      })
      .joining((user) => {
        this.users.push(user);
      })
      .leaving((user) => {
        this.users = this.users.filter((u) => u.id != user.id);
      })
      .listen("MessageSent", (event) => {
        this.messages.push(event.message);
      })
      .listenForWhisper("typing", (response) => {
        this.activeUser = response;

        if (this.typingTimer) {
          clearTimeout(this.typingTimer);
        }

        this.typingTimer = setTimeout(() => {
          this.activeUser = "";
        }, 3000);
      });
  },
  methods: {
    fetchMessages() {
      axios.get("messages").then((response) => {
        this.messages = response.data;
      });
    },
    sendMessage() {
      axios
        .post("messages", { message: this.message })
        .then((response) => {
          this.messages.push({
            message: this.message,
            user: this.user,
          });
          this.message = "";
        })
        .catch((error) => {
          console.log(error.response.data);
        });
    },
    sendTypingEvent() {
      Echo.join("chat").whisper("typing", this.user);
    },
  },
};
</script>

<style scoped>
.rightMsg {
  text-align: right;
  color: #6875f5;
}
</style>
