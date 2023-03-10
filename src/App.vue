<template>
  <div class="chat-container">
    <div class="chat-header">Chat App</div>
    <div class="chat-messages">
      <div v-for="message in messages" :key="message.id" class="message">
        <div v-if="message.type === 'received'" class="received-message">
          {{ message.content }}
        </div>
        <div v-else-if="message.type === 'sent'" class="sent-message">
          {{ message.content }}
        </div>
      </div>
    </div>
    <div class="chat-input">
      <input
        v-model="inputMessage"
        type="text"
        placeholder="输入你的信息…"
      />
      <button @click="sendMessage">Send</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import axios from "axios";

interface Message {
  id: number;
  type: string;
  content: string;
}

const API_ENDPOINT =
  "https://api.openai.com/v1/engine/davinci-codex/completions";

const apikey = localStorage.getItem("apikey") || "";
if (!apikey) {
  const inputKey = prompt("请输入您的OpenAI API密钥:");
  if (inputKey) {
    localStorage.setItem("apikey", inputKey);
  }
}

const messages = ref<Message[]>([]);
const inputMessage = ref<string>("");

const sendMessage = async () => {
  const apiKey = localStorage.getItem("apikey");
  if (!apiKey) {
    alert("OpenAI API key is not found!");
    return;
  }

  const promptText = `User: ${inputMessage.value}`;
  const requestBody = {
    prompt: promptText,
    max_tokens: 60,
    n: 1,
    stop: ["\n"],
  };

  try {
    const response = await axios.post(API_ENDPOINT, requestBody, {
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${apiKey}`,
      },
    });

    const botMessage = response.data.choices[0].text.trim();
    messages.value.push(
      {
        id: Date.now(),
        type: "sent",
        content: inputMessage.value,
      },
      {
        id: Date.now(),
        type: "received",
        content: botMessage,
      }
    );
    inputMessage.value = "";
  } catch (error) {
    console.error(error);
  }
};

onMounted(() => {
  const welcomeMessage = "你好，我是你的聊天机器人!今天我能为您效劳吗?";
  messages.value.push({
    id: Date.now(),
    type: "received",
    content: welcomeMessage,
  });
});
</script>

<style scoped>
.chat-container {
  width: 400px;
  height: 600px;
  margin: 0 auto;
  border: 1px solid #ccc;
  border-radius: 5px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.chat-header {
  background-color: #41b883;
  color: #fff;
  font-size: 24px;
  font-weight: bold;
  text-align: center;
  padding: 10px;
}

.chat-messages {
  flex: 1;
  overflow-y: scroll;
  padding: 10px;
}

.message {
  margin-bottom: 10px;
}

.received-message {
  background-color: #eee;
  padding: 5px;
  border-radius: 5px;
  text-align: left;
}

.sent-message {
  background-color: #fff;
  padding: 5px;
  border-radius: 5px;
  text-align: right;
}

.chat-input {
  display: flex;
  align-items: center;
  padding: 10px;
  background-color: #f1f1f1;
}

.chat-input input {
  flex: 1;
  padding: 5px;
  border: none;
  border-radius: 5px;
  margin-right: 10px;
}

.chat-input button {
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  background-color: #41b883;
  color: #fff;
  cursor: pointer;
}
</style>
