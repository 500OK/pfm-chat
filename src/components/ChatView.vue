<script setup>
import { ref, nextTick } from "vue";
import { marked } from "marked"; // Import the marked library

const BACK_END_URL = "https://mosquito-golden-silkworm.ngrok-free.app";

const userInput = ref("");
const messages = ref([
  { text: "Hello! **How** can I _help_ you?", sender: "bot" }, // Sample markdown message
  { text: "What can you do?", sender: "user" },
]);

const predefinedPrompts = ref({
  anomaly_detection_agent: "Suspicious transactions",
  budget_prediction_agent: "Budget predictions",
  recommendation_engine_agent: "Reccomentations",
  transaction_categorization_agent: "Transaction categorisation",
});

const messagesContainer = ref(null);
const isLoading = ref(false); // Flag to indicate loading

// Send message function
const sendMessage = async () => {
  if (userInput.value.trim() !== "") {
    messages.value.push({ text: userInput.value, sender: "user" });
    const userPrompt = userInput.value;

    userInput.value = "";

    await nextTick();
    scrollToBottom();

    isLoading.value = true;
    messages.value.push({ text: "", sender: "bot", isLoading: true });

    await nextTick();
    scrollToBottom();

    try {
      const response = await fetch(`${BACK_END_URL}/generate`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          user_id: "0d7051n1",
          prompt: userPrompt,
        }),
      });

      const result = await response.json();

      // Check if the result contains the 'response' key and it's an object
      if (result && result.response && typeof result.response === "object") {
        // Remove the loading indicator
        messages.value.pop();

        // Add a message for each agent in the response object
        Object.entries(result.response).forEach(([agent, responseText]) => {
          messages.value.push({
            text: `${agent}: ${responseText}`,
            sender: "bot",
          });
        });
      } else {
        // Default handling if 'response' is not structured as expected
        messages.value[messages.value.length - 1] = {
          text: result.response || "I can answer questions!",
          sender: "bot",
        };
      }
    } catch (error) {
      console.error("Error in POST request:", error);
      messages.value[messages.value.length - 1] = {
        text: "Something went wrong. Please try again.",
        sender: "bot",
      };
    }

    isLoading.value = false;
    await nextTick();
    scrollToBottom();
  }
};

const sendPredefinedMessage = async (key, prompt) => {
  messages.value.push({ text: prompt, sender: "user" });
  await nextTick();
  scrollToBottom();

  isLoading.value = true;
  messages.value.push({ text: "", sender: "bot", isLoading: true });
  await nextTick();
  scrollToBottom();

  try {
    const response = await fetch(`${BACK_END_URL}/generate`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        user_id: "0d7051n1",
        prompt: key,
      }),
    });

    const result = await response.json();

    if (result && result.response && typeof result.response === "object") {
      // Remove the loading indicator
      messages.value.pop();

      // Add a message for each agent in the response object
      Object.entries(result.response).forEach(([agent, responseText]) => {
        messages.value.push({
          text: `${agent}: ${responseText}`,
          sender: "bot",
        });
      });
    } else {
      // Default handling if 'response' is not structured as expected
      messages.value[messages.value.length - 1] = {
        text: result.response || "I can answer questions!",
        sender: "bot",
      };
    }
  } catch (error) {
    console.error("Error in POST request:", error);
    messages.value[messages.value.length - 1] = {
      text: "Something went wrong. Please try again.",
      sender: "bot",
    };
  }

  isLoading.value = false;
  await nextTick();
  scrollToBottom();
};

const scrollToBottom = () => {
  if (messagesContainer.value) {
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight;
  }
};

const autoResizeTextarea = (event) => {
  const textarea = event.target;
  textarea.style.height = "auto";
  textarea.style.height = `${textarea.scrollHeight}px`;
};
</script>

<template>
  <div class="flex flex-col h-screen bg-gray-50 text-gray-900">
    <!-- Chat messages section -->
    <div class="flex-1 overflow-auto p-6 space-y-4" ref="messagesContainer">
      <div v-for="(message, index) in messages" :key="index" class="flex flex-col">
        <div :class="[
          'max-w-xs p-4 rounded-lg shadow-sm transition-all overflow-x-scroll pb-1',
          message.sender === 'user'
            ? 'bg-blue-600 text-white ml-auto'
            : 'bg-gray-100 text-gray-900',
        ]">
          <!-- Show spinner when loading -->
          <div v-if="message.isLoading" class="flex items-center justify-start space-x-2">
            <div class="w-4 h-4 border-4 border-t-transparent border-blue-600 rounded-full animate-spin"></div>
            <span class="text-gray-500">Thinking...</span>
          </div>
          <!-- Show actual message text when not loading -->
          <span v-else v-html="marked(message.text)"></span>
          <!-- Render Markdown content -->
        </div>
      </div>
    </div>

    <!-- Predefined prompts section -->
    <div class="p-4 bg-white border-t border-gray-200">
      <div class="flex flex-wrap gap-2">
        <button v-for="([key, prompt], index) in Object.entries(predefinedPrompts)" :key="index"
          @click="sendPredefinedMessage(key, prompt)"
          class="px-4 py-2 bg-gray-100 rounded-lg text-gray-800 hover:bg-gray-200 transition-all focus:outline-none focus:ring-2 focus:ring-blue-400">
          {{ prompt }}
        </button>
      </div>
    </div>

    <!-- Input field for user message (multiline) -->
    <div class="flex items-center p-4 bg-white border-t border-gray-200">
      <textarea v-model="userInput" @keydown.enter.exact.prevent="sendMessage" @input="autoResizeTextarea"
        placeholder="Type a message..." rows="1"
        class="flex-1 p-3 bg-gray-100 rounded-md border border-gray-300 focus:outline-none focus:ring-2 focus:ring-soft-neon-blue focus:bg-white transition-all resize-none"></textarea>
      <button @click="sendMessage"
        class="ml-4 px-5 py-3 bg-blue-600 text-white rounded-md shadow-sm hover:bg-blue-500 transition-all focus:outline-none focus:ring-2 focus:ring-blue-400">
        Send
      </button>
    </div>
  </div>
</template>

<style scoped>
/* Custom scrollbar styling */
.flex-1 {
  min-height: 0;
}

::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-thumb {
  background-color: rgba(0, 0, 0, 0.1);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background-color: rgba(0, 0, 0, 0.2);
}

/* Spinner animation */
.animate-spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}
</style>
