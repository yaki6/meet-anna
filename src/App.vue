<template lang="pug">
#app
  img(alt="Meet Anna", src="./assets/logo.png", height="200")
  VueBotUI(
    :options="botOptions",
    :messages="messageData",
    :bot-typing="botTyping",
    :input-disable="inputDisable",
    :is-open="false",
    @init="botStart",
    @msg-send="msgSend"
  )
  p  messageData: {{ messageData }}
</template>
<script>
import BotIcon from "./assets/icons/bot.png";
import { VueBotUI } from "./vue-bot-ui";
// import { messageService } from "./helpers/message";
import axios from 'axios';
// import { useStorage } from '@vueuse/core'
const apiKey = process.env.VUE_APP_OPENAI_API_KEY;
const client = axios.create({
  headers: {
    Authorization: "Bearer " + apiKey
  }
});

export default {
  components: {
    BotIcon,
    VueBotUI
  },

  data () {
    return {
      messageData: [],
      botTyping: false,
      inputDisable: false,
      botOptions: {
        botAvatarImg: BotIcon,
        boardContentBg: "#f4f4f4",
        msgBubbleBgBot: "#fff",
        inputPlaceholder: "Type hereeee..",
        inputDisableBg: "#fff",
        inputDisablePlaceholder: "Hit the buttons above to respond"
      }
    };
  },

  methods: {
    botStart () {
      // Request first message here
      if (!this.messageData) {
        this.botTyping = true;
        setTimeout(() => {
          this.botTyping = false;
          this.messageData.push({
            agent: "bot",
            type: "text",
            text: "Hello, this is Anna. What do you want to talk about?"
          });
        }, 500);
      }
      // Fake typing for the first message
    },

    msgSend (value) {
      // Push the user's message to board
      this.messageData.push({
        agent: "user",
        type: "text",
        text: value.text
      });

      this.getResponse(value.text);
    },

    // Submit the message from user to open API, then get the response from Bot
    getResponse (text) {
      // Loading
      this.botTyping = true;

      // Post the message from user here
      // Then get the response as below
      const params = {
        prompt: text,
        model: "text-davinci-003",
        max_tokens: 100,
        temperature: 0.7
      };
      // Create new message from openAI API
      client
        .post("https://api.openai.com/v1/completions", params)
        .then((result) => {
          const response = result.data.choices[0].text
          const replyMessage = {
            agent: "bot",
            type: "text",
            text: response
          };
          this.inputDisable = response.disableInput;
          this.messageData.push(replyMessage);
          // finish
          this.botTyping = false;
        })
        .catch(() => {
        });
    }
  },
  watch: {
    messageData () {
      const parsed = JSON.stringify(this.messageData);
      localStorage.setItem('messageData', parsed);
    }
  },
  mounted () {
    if (localStorage.getItem('messageData')) {
      try {
        this.messageData = JSON.parse(localStorage.getItem('messageData'));
      } catch (e) {
        localStorage.removeItem('messageData');
      }
    }
  }
};
</script>
<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  // text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
