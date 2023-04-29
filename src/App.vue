<template>
  <v-app>
    <v-main class="py-10">
      <v-container>
        <div>
          <v-row>
            <v-col
                cols="12"
                sm="5"
                class="px-0 py-0"
            >
              <v-sheet
                  height="85vh"
                  style="background-color: #1E1E1E; position: relative"
              >
                <div class="message-wrapper">
                  <header class="user-info">
                    <div class="user-info-wrapper">
                      <div class="personal-info">
                        <h4 class="user-info-name"  >
                          {{ "Caronas Taquara x Fundão" }}
                        </h4>
                        <p class="last-seen">{{userInfo.lastAct}}</p>
                      </div>
                    </div>
                  </header>
                  <main class="message-area">
                    <div class="main-wrapper">
                      <ul class="msg-list">
                        <li v-for="item in items" :key="item.message" class="chat-msg msg-receive" v-html="item.message">

                          <i class="icon-check_all_big"></i>
                          <span class="close-button">x</span>
                        </li>
                        <li
                            class="chat-msg msg-sent"
                            v-for="(message) in filteredMessages"
                            :key="message.id"
                        >
                          {{message.message}}
                          <span class="message-time">{{message.time | filterDateTime}}</span>
                          <i class="icon-check_all_big"></i>
                          <span class="close-button" @click="deleteMessage(message.id)">x</span>
                        </li>
                      </ul>
                    </div>
                  </main>
                  <footer class="user-msg-footer">
                    <div class="file-input-wrapper">
                      <input type="file" id="file-input" class="file-input">
                      <label for="file-input" class="input-label">
                        <i class="icon-slider_02"></i>
                      </label>
                    </div>
                    <div class="msg-input-wrapper">
        <textarea class="input-message"
                  placeholder="Write a message..."
                  v-model.trim="createdMessage"
                  @keypress.enter="postMessage"
        ></textarea>
                    </div>
                    <div class="footer-widgets-wrapper">
                      <div class="footer-icons">
                        <i class="icon-happy"></i>
                        <i class="icon-mic"></i>
                      </div>
                    </div>
                  </footer>

                  <v-dialog
                      v-model="dialog"
                      max-width="400px"
                  >
                    <v-card>
                      <user-info :userInfo="userInfo" />
                    </v-card>
                  </v-dialog>
                </div>
              </v-sheet>
            </v-col>
          </v-row>
        </div>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>

import UserInfo from "@/components/users/UserInfo";

export default {
  components: {UserInfo},
  data: () => ({
    drawer: false,
    group: null,
    searchArea: null,
    tabItems: [
      {name: 'All', icon: 'icon-chat', notification: 2},
      {name: 'Users', icon: 'icon-user', notification: 3}
    ],
    dialog: false,
    createdMessage: '',
    messages: [],
    filteredMessages: [],
    userId: 1,
    userInfo: {
      fullName: '',
      photoUrl: '',
      userName: '',
      phoneNumber: '',
      bio: '',
      lastAct: ''
    },
    starting_0: true,
    starting_1: false,
    starting_2: false,
    items: []

  }),
  methods: {

    start() {
      const user_id = localStorage.getItem('user_id');
      const username = localStorage.getItem('username')

      if (user_id && username) {
        this.starting_0 = false
        this.starting_1 = false
        this.starting_2 = false
        this.items.push({ message: 'Digite o comando. Para ajuda, digite /help'})
      }
      else if (!username) {
        this.items.push({ message: 'Digite o nome de usuário do Telegram'})
        this.starting_1 = true
      }
      else if (!user_id) {
        this.items.push({ message: 'Digite seu número de celular'})
        this.starting_2 = true
      }
    },

    async postMessage() {
      console.log(this.createdMessage)
      if (this.createdMessage.length > 0) {
        console.log(this.starting_1)
        if (this.starting_1) {
          localStorage.setItem('username', this.createdMessage)
          this.starting_1 = false

          this.filteredMessages.push({message: this.createdMessage});

          this.createdMessage = ''
          this.start()
        }

        else if (this.starting_2) {
          localStorage.setItem('user_id', this.createdMessage)
          this.starting_2 = false

          this.filteredMessages.push({message: this.createdMessage});

          this.createdMessage = ''
          this.start()
        }


        if (!this.starting_0) {

          const messages = {
            message: {
              from: {
                username: localStorage.getItem('username'),
                id: localStorage.getItem('user_id'),
              },
              chat: {
                id: "-1001104177248"
              },
              text: this.createdMessage
            }
          }

          const response_a = this.$http.post('https://caronastaquarabot-newstack.herokuapp.com/', messages)
              .then(response => {
                return response.text()
              })

          console.log(response_a)

          this.createdMessage = ''

          this.items.push({message: await response_a})
        }
      }
    },
    getNow: function() {
      let timestamp = '';
      const today = new Date();
      const date = today.getFullYear()+'-'+(today.getMonth()+1)+'-'+today.getDate();
      const time = today.getHours() + ":" + today.getMinutes() + ":" + today.getSeconds();
      timestamp = date + ' ' + time;
      console.log(timestamp)
      return today
    },
    getMessageFromUser(id) {
      this.userId = id;
      console.log('from app', id)
      this.filteredMessages = this.messages.filter(function (message) {
        return message.chat_id == id
      })
    },
  },
  created() {
    this.start()
  },
};
</script>

<style>
@import 'assets/css/style.css';
@import 'assets/fonts/icons/style.css';

</style>
