<template>
  <div class="chat-container">
    <div class="chat-messages" ref="messagesContainer">
      <div v-for="(message, index) in messages" :key="index" 
           :class="['message', message.role]">
        <div class="message-content">
          <div class="avatar">
            <i :class="['icon', message.role === 'user' ? 'fas fa-user' : 'fas fa-robot']"></i>
          </div>
          <div class="message-bubble">
            <div class="text" :class="{ 'streaming': message.isStreaming }" v-html="renderMessage(message.content)"></div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="input-container">
      <div class="input-wrapper">
        <textarea 
          v-model="userInput" 
          @keydown.enter.prevent="sendMessage"
          placeholder="Type your message here..."
          rows="1"
          ref="inputField"
          :disabled="isLoading"
        ></textarea>
        <button @click="sendMessage" :disabled="isLoading" class="send-button">
          <i class="fas fa-paper-plane"></i>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick } from 'vue'
import OpenAI from 'openai'
import { marked } from 'marked'
import DOMPurify from 'dompurify'

// Configure marked
marked.setOptions({
  breaks: true,
  gfm: true
})

const renderMessage = (content: string): string => {
  try {
    const parsed = marked.parse(content)
    return typeof parsed === 'string' ? DOMPurify.sanitize(parsed) : content
  } catch (e) {
    console.error('Markdown parsing error:', e)
    return content
  }
}

const apiKey = import.meta.env.VITE_OPENAI_API_KEY

if (!apiKey) {
  console.error('OpenAI API key is not set in .env file')
}

const openai = new OpenAI({
  apiKey: apiKey,
  dangerouslyAllowBrowser: true
})

interface Message {
  role: 'user' | 'assistant'
  content: string
  isStreaming?: boolean
}

const messages = ref<Message[]>([])
const userInput = ref('')
const isLoading = ref(false)
const messagesContainer = ref<HTMLElement | null>(null)
const inputField = ref<HTMLTextAreaElement | null>(null)
const streamingMessageId = ref<number | null>(null)

const sendMessage = async () => {
  if (!userInput.value.trim() || isLoading.value) return
  if (!apiKey) {
    messages.value.push({
      role: 'assistant',
      content: 'Error: OpenAI API key is not set. Please add your API key to the .env file.'
    })
    return
  }

  // Add user message
  const userMessage = {
    role: 'user' as const,
    content: userInput.value.trim()
  }
  messages.value.push(userMessage)

  // Create placeholder for assistant's response
  const assistantMessageIndex = messages.value.length
  messages.value.push({
    role: 'assistant',
    content: '',
    isStreaming: true
  })
  streamingMessageId.value = assistantMessageIndex

  userInput.value = ''
  isLoading.value = true

  try {
    const stream = await openai.chat.completions.create({
      messages: [
        { role: 'system', content: 'You are a helpful assistant.' },
        ...messages.value.slice(0, -1).map(msg => ({ role: msg.role, content: msg.content }))
      ],
      model: 'gpt-3.5-turbo',
      stream: true,
    })

    let content = ''
    for await (const chunk of stream) {
      const text = chunk.choices[0]?.delta?.content || ''
      content += text
      if (streamingMessageId.value !== null) {
        messages.value[streamingMessageId.value].content = content
      }
      await nextTick()
      scrollToBottom()
    }

    // Update final message
    if (streamingMessageId.value !== null) {
      messages.value[streamingMessageId.value].isStreaming = false
    }
  } catch (error) {
    console.error('Error:', error)
    if (streamingMessageId.value !== null) {
      messages.value[streamingMessageId.value] = {
        role: 'assistant',
        content: 'Sorry, I encountered an error. Please try again.'
      }
    }
  } finally {
    isLoading.value = false
    streamingMessageId.value = null
    await nextTick()
    scrollToBottom()
    // Refocus the input field
    if (inputField.value) {
      inputField.value.focus()
    }
  }
}

const scrollToBottom = () => {
  if (messagesContainer.value) {
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
  }
}

onMounted(() => {
  if (inputField.value) {
    inputField.value.focus()
  }
})
</script>

<style scoped>
.chat-container {
  display: flex;
  flex-direction: column;
  height: calc(100vh - 4rem);
  width: 100%;
  background: var(--background-color);
  overflow: hidden;
  position: relative;
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 0 5%;
  scroll-behavior: smooth;
  -webkit-overflow-scrolling: touch;
}

.message {
  margin-bottom: 1.5rem;
  opacity: 0;
  transform: translateY(20px);
  animation: messageAppear 0.3s ease forwards;
}

@keyframes messageAppear {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.message-content {
  display: flex;
  align-items: flex-start;
  gap: 1rem;
  max-width: 80%;
  transition: all 0.3s ease;
}

.message.user .message-content {
  margin-left: auto;
  flex-direction: row-reverse;
}

.avatar {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  flex-shrink: 0;
  transition: transform 0.3s ease;
}

.avatar:hover {
  transform: scale(1.1);
}

.message.assistant .avatar {
  background: var(--success-color);
  color: white;
}

.message.user .avatar {
  background: var(--primary-color);
  color: white;
}

.message-bubble {
  flex: 1;
  max-width: calc(100% - 50px);
}

.text {
  padding: 1rem 1.25rem;
  border-radius: 16px;
  font-size: 0.9375rem;
  line-height: 1.6;
  white-space: pre-wrap;
  transition: all 0.2s ease;
}

.message.assistant .text {
  background: var(--surface-color);
  color: var(--text-primary);
  border: 1px solid var(--border-color);
}

.message.user .text {
  background: var(--primary-color);
  color: var(--surface-color);
}

.input-container {
  border-top: 1px solid var(--border-color);
  padding: 1rem;
  background: var(--background-color);
  position: relative;
  width: 100%;
}

.input-wrapper {
  display: flex;
  gap: 0.75rem;
  background: var(--surface-color);
  padding: 1rem;
  border-radius: 0.75rem;
  border: 1px solid var(--border-color);
  transition: all 0.2s ease;
  width: 95%;
  max-width: 1200px;
  margin: 0 auto;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
}

.input-wrapper:focus-within {
  border-color: var(--primary-color);
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
}

textarea {
  flex: 1;
  padding: 0;
  background: transparent;
  border: none;
  resize: none;
  font-family: inherit;
  font-size: 1rem;
  line-height: 1.5;
  color: var(--text-primary);
  min-height: 24px;
  max-height: 200px;
  overflow-y: auto;
  scrollbar-width: none;
  -ms-overflow-style: none;
  transition: all 0.2s ease;
}

textarea::-webkit-scrollbar {
  display: none;
}

textarea:focus {
  outline: none;
}

textarea::placeholder {
  color: var(--text-secondary);
}

.send-button {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--primary-color);
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
  overflow: hidden;
  align-self: flex-end;
}

.send-button::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0.2);
  transform: translate(-50%, -50%) scale(0);
  border-radius: 50%;
  transition: transform 0.3s ease;
}

.send-button:hover::after {
  transform: translate(-50%, -50%) scale(2);
}

.send-button:hover {
  background: var(--primary-dark);
  transform: translateY(-1px);
}

.send-button:active {
  transform: translateY(1px);
}

.send-button:disabled {
  background: var(--text-secondary);
  cursor: not-allowed;
  transform: none;
}

.send-button i {
  font-size: 0.875rem;
}

/* Media Queries */
@media (min-width: 1920px) {
  .chat-messages {
    padding: 0 5%;
  }

  .input-container {
    padding: 1.5rem 2rem;
  }

  .input-wrapper {
    width: min(100%, 75vw);
    max-width: 60rem;
  }
}

@media (min-width: 1440px) and (max-width: 1919px) {
  .chat-messages {
    padding: 0 5%;
  }

  .input-container {
    padding: 1.5rem 2rem;
  }

  .input-wrapper {
    width: min(100%, 80vw);
    max-width: 60rem;
  }
}

@media (min-width: 1200px) and (max-width: 1439px) {
  .chat-messages {
    padding: 0 5%;
  }

  .input-container {
    padding: 1.5rem 2rem;
  }

  .input-wrapper {
    width: min(100%, 85vw);
    max-width: 56rem;
  }
}

@media (min-width: 768px) and (max-width: 1199px) {
  .chat-messages {
    padding: 0 3%;
  }

  .input-container {
    padding: 1.5rem 1rem;
  }

  .input-wrapper {
    width: min(100%, 90vw);
    max-width: 52rem;
    padding: 1rem;
  }
}

@media (max-width: 768px) {
  .chat-messages {
    padding: 0 1rem;
  }

  .input-container {
    padding: 0.75rem 0.5rem;
  }

  .input-wrapper {
    width: 95%;
    padding: 0.75rem;
    margin: 0;
    border-radius: 0.75rem;
  }

  textarea {
    font-size: 1rem;
  }

  .send-button {
    width: 32px;
    height: 32px;
  }
}

@media (max-width: 480px) {
  .chat-messages {
    padding: 0 0.75rem;
  }

  .input-container {
    padding: 0.75rem 0.25rem;
  }

  .input-wrapper {
    width: 98%;
    padding: 0.625rem;
  }

  textarea {
    font-size: 0.9375rem;
  }
}

/* iOS Safari fixes */
@supports (-webkit-touch-callout: none) {
  .chat-container {
    height: calc(100vh - 4rem - env(safe-area-inset-bottom));
  }
}

.streaming {
  position: relative;
}

.cursor {
  display: inline-block;
  width: 2px;
  height: 1em;
  background: currentColor;
  margin-left: 2px;
  animation: blink 1s step-end infinite;
}

@keyframes blink {
  from, to {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}

textarea:disabled {
  cursor: not-allowed;
  opacity: 0.7;
}
</style> 