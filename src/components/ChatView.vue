<script setup>
import { ref } from "vue";

const userInput = ref("");
const messages = ref([
  { text: "Hello! How can I help you?", sender: "bot" },
  { text: "What can you do?", sender: "user" },
]);

const sendMessage = () => {
  if (userInput.value.trim() !== "") {
    messages.value.push({ text: userInput.value, sender: "user" });
    userInput.value = "";
    // Simulate bot reply
    setTimeout(() => {
      messages.value.push({ text: "I can answer questions!", sender: "bot" });
    }, 1000);
  }
};
</script>

<template>
  <div class="flex flex-col h-screen bg-gray-100">
    <!-- Chat messages section -->
    <div class="flex-1 overflow-auto p-4 space-y-4">
      <div v-for="(message, index) in messages" :key="index" class="flex">
        <div :class="[
          'max-w-xs p-3 rounded-lg',
          message.sender === 'user'
            ? 'bg-blue-500 text-white self-end'
            : 'bg-gray-200 text-gray-800',
        ]">
          {{ message.text }}
        </div>
      </div>
    </div>

    <!-- Input field for user message -->
    <div class="flex items-center p-4 bg-white border-t border-gray-300">
      <input v-model="userInput" @keydown.enter="sendMessage" type="text" placeholder="Type a message..."
        class="flex-1 p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500" />
      <button @click="sendMessage" class="ml-4 px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600">
        Send
      </button>
    </div>
  </div>
</template>

<style scoped>
/* Optional custom styles */
</style>
