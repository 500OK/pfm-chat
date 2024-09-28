<script setup>
import { ref, nextTick } from "vue";

const userInput = ref("");
const messages = ref([
  { text: "Hello! How can I help you?", sender: "bot" },
  { text: "What can you do?", sender: "user" },
]);

const predefinedPrompts = ref([
  "Tell me a joke",
  "What’s the weather like?",
  "Help with my account",
  "What services do you offer?",
]);

const messagesContainer = ref(null);

const sendMessage = async () => {
  if (userInput.value.trim() !== "") {
    messages.value.push({ text: userInput.value, sender: "user" });
    userInput.value = "";

    // Wait for DOM updates, then scroll to the bottom
    await nextTick();
    scrollToBottom();

    // Simulate bot reply
    setTimeout(async () => {
      messages.value.push({ text: "I can answer questions!", sender: "bot" });

      await nextTick();
      scrollToBottom();
    }, 1000);
  }
};

const sendPredefinedMessage = async (prompt) => {
  messages.value.push({ text: prompt, sender: "user" });
  await nextTick();
  scrollToBottom();

  // Simulate bot reply after prompt is selected
  setTimeout(async () => {
    messages.value.push({ text: `You asked: ${prompt}`, sender: "bot" });
    await nextTick();
    scrollToBottom();
  }, 1000);
};

const scrollToBottom = () => {
  if (messagesContainer.value) {
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight;
  }
};
</script>

<template>
  <div class="flex flex-col h-screen bg-gray-100">
    <!-- Chat messages section -->
    <div class="flex-1 overflow-auto p-4 space-y-4" ref="messagesContainer">
      <div v-for="(message, index) in messages" :key="index" class="flex flex-col">
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

    <!-- Predefined prompts section -->
    <div class="p-4 bg-white border-t border-gray-300">
      <div class="flex space-x-2">
        <button v-for="(prompt, index) in predefinedPrompts" :key="index" @click="sendPredefinedMessage(prompt)"
          class="px-4 py-2 bg-gray-200 rounded-lg hover:bg-gray-300 text-gray-800">
          {{ prompt }}
        </button>
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
