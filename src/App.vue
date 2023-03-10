<template>
  <div class="chat-app" @click="handleBlur">
    <div class="title">Chat with AI</div>
    <div class="message-list">
      <div class="message" v-for="(message, index) in messages" :key="index">
        <div class="message-sender" :class="message.sender.toLowerCase()">
          {{ message.sender }}
        </div>
        <div class="message-text">{{ message.text }}</div>
        <div class="message-tokens" v-if="message.tokens">
          本次提问消耗tokens:{{ message.tokens }}
        </div>
      </div>
    </div>
    <div class="input-container">
      <input
        ref="inputRef"
        v-model="inputMessage"
        placeholder="Type a message..."
      />
      <button @click="sendMessage">Send</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import axios from "axios";

const inputRef = ref<any>(null);

type Message = {
  sender: string;
  text: string;
  tokens?: string;
};

const API_URL = "https://api.openai.com/v1/chat/completions";

const messages = ref<Message[]>([]);
const inputMessage = ref("");

const getApiKey = (): string | null => {
  const apiKey = localStorage.getItem("OPENAI_API_KEY");
  if (!apiKey) {
    const input = prompt("Please enter your OpenAI API key:");
    if (input) {
      localStorage.setItem("OPENAI_API_KEY", input);
      return input;
    } else {
      return null;
    }
  }
  return apiKey;
};

const sendMessage = async () => {
  const apiKey = getApiKey();
  if (!apiKey) return;
  const message = {
    sender: "User",
    text: inputMessage.value,
  };
  messages.value.push(message);
  axios
    .post(
      API_URL,
      {
        messages: [{ role: "user", content: inputMessage.value }],
        model: "gpt-3.5-turbo",
        temperature: 0.7,
      },
      {
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${apiKey}`,
        },
      }
    )
    .then((response) => {
      const responseMessage = {
        sender: "ChatBot",
        text: response.data.choices[0].message.content,
        tokens: response.data.usage.total_tokens,
      };

      messages.value.push(responseMessage);
    })
    .catch((error) => {
      console.log(error);
    });
  inputMessage.value = "";
};

onMounted(() => {
  getApiKey();
});

function handleBlur() {
  inputRef.value!.blur();
}
</script>

<style scoped>
.chat-app {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.title {
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #41b883;
  color: #fff;
  padding: 10px;
}

.message-list {
  flex: 1;
  overflow: auto;
  padding: 10px;
}

.message {
  margin-bottom: 10px;
}

.message-sender {
  font-weight: bold;
  margin-bottom: 5px;
}

.user {
  color: #41b883;
}

.chatbot {
  color: #2c3e50;
}

.input-container {
  display: flex;
  align-items: center;
  padding: 10px;
  background-color: #f5f5f5;
}

.message-tokens {
  margin-top: 6px;
  font-size: 12px;
  color: rgba(0, 0, 0, 0.3);
}

input {
  flex: 1;
  padding: 10px;
  border-radius: 5px;
  border: none;
  margin-right: 10px;
}

button {
  background-color: #41b883;
  color: #fff;
  border: none;
  padding: 10px;
  border-radius: 5px;
  cursor: pointer;
}
</style>
