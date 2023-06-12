<script setup lang="ts">
import { ref, reactive } from 'vue'
import linkifyStr from 'linkify-string'

interface Message {
  text: string
  type: string
}

const messages = reactive<Message[]>([])
const inputMessage = ref('')
const historySummary = ref('')
const isLoading = ref(false)

async function sendChatMessage(event: Event) {
  event.preventDefault()
  try {
    isLoading.value = true
    if (!inputMessage.value) {
      isLoading.value = false
      return
    }
    const input = inputMessage.value.trim()
    addMessage(inputMessage.value, 'user')
    inputMessage.value = ''
    const response = await fetch(import.meta.env.VITE_BACKEND_URL, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        inputMessage: input,
        historySummary: historySummary.value
      })
    })
    const data = await response.json()
    const { text, chat_history } = data
    inputMessage.value = ''
    historySummary.value = chat_history
    addMessage(text, 'assistant')
    isLoading.value = false
  } catch (error) {
    isLoading.value = false
    console.error(error)
  }
}

function addMessage(message: string, type: string) {
  messages.push({
    text: linkifyStr(message, { target: '_blank' }),
    type
  })
}
</script>

<template>
  <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
    <div class="mx-auto max-w-3xl h-[100vh] flex flex-col justify-end py-4">
      <div v-for="(message, index) in messages" :key="index" class="inset-x-0 pb-4">
        <div
          class="rounded-xl p-6 shadow-lg"
          :class="[
            message.type === 'user' && 'bg-blue-600',
            message.type === 'assistant' && 'bg-white ring-1 ring-blue-700'
          ]"
        >
          <p
            v-html="message.text"
            class="text-sm leading-6 [&_a]:font-bold [&_a]:text-blue-800"
            :class="[
              message.type === 'user' && 'text-white',
              message.type === 'assistant' && 'text-blue-700'
            ]"
          ></p>
        </div>
      </div>
      <form @submit="sendChatMessage" class="relative">
        <div
          class="overflow-hidden rounded-lg pb-12 shadow-sm ring-1 ring-inset ring-blue-700 focus-within:ring-2 focus-within:ring-blue-500"
        >
          <textarea
            v-model="inputMessage"
            rows="2"
            name="comment"
            id="comment"
            :disabled="isLoading"
            class="block w-full resize-none border-0 bg-transparent py-1.5 text-blue-700 placeholder:text-blue-300 focus:ring-0 sm:text-sm sm:leading-6"
            :class="[isLoading && 'cursor-not-allowed']"
            placeholder="Add your question..."
          />
        </div>

        <div class="absolute inset-x-0 bottom-0 flex justify-between py-2 pl-3 pr-2">
          <div class="flex items-center space-x-5">
            <div class="flex items-center"></div>
            <div class="flex items-center"></div>
          </div>
          <button
            :disabled="isLoading"
            type="submit"
            class="rounded-md bg-white px-2.5 py-1.5 text-sm font-semibold text-blue-700 shadow-sm ring-1 ring-inset ring-blue-700 hover:bg-blue-500-50 inline-flex gap-2"
            :class="[isLoading && 'cursor-not-allowed']"
          >
            <svg
              v-if="isLoading"
              class="animate-spin h-5 w-5 text-blue-700"
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
            >
              <circle
                class="opacity-25"
                cx="12"
                cy="12"
                r="10"
                stroke="currentColor"
                stroke-width="4"
              ></circle>
              <path
                class="opacity-75"
                fill="currentColor"
                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
              ></path>
            </svg>
            Send
          </button>
        </div>
      </form>
    </div>
  </div>
</template>
