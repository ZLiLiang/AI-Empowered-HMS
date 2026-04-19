<script setup>
import { onBeforeUnmount, ref } from 'vue'

const fileInputRef = ref(null)
const uploadedFileName = ref('')
const previewUrl = ref('')
const isUploaded = ref(false)
const isThinking = ref(false)
const isScreeningFinished = ref(false)
const answerInput = ref('')

const questions = [
  'Please summarize one project where you used Java and Spring Boot to solve a high-concurrency challenge.',
  'How did you handle a production incident, and what preventive improvement did you implement afterward?'
]

const currentQuestionIndex = ref(-1)
const conversation = ref([])

const wait = (ms) => new Promise((resolve) => setTimeout(resolve, ms))

const openUploadDialog = () => {
  fileInputRef.value?.click()
}

const resetFlow = () => {
  conversation.value = []
  currentQuestionIndex.value = -1
  answerInput.value = ''
  isScreeningFinished.value = false
}

const askNextQuestion = async () => {
  if (currentQuestionIndex.value >= questions.length - 1) {
    return
  }

  isThinking.value = true
  await wait(1000)
  isThinking.value = false

  currentQuestionIndex.value += 1
  conversation.value.push({
    role: 'assistant',
    text: questions[currentQuestionIndex.value]
  })
}

const onFileChange = async (event) => {
  const file = event.target.files?.[0]
  if (!file) return

  if (previewUrl.value) {
    URL.revokeObjectURL(previewUrl.value)
  }

  uploadedFileName.value = file.name
  previewUrl.value = URL.createObjectURL(file)
  isUploaded.value = true

  resetFlow()
  await askNextQuestion()
}

const submitAnswer = async () => {
  const text = answerInput.value.trim()
  if (!text || isThinking.value || !isUploaded.value) return

  conversation.value.push({ role: 'user', text })
  answerInput.value = ''

  if (currentQuestionIndex.value < questions.length - 1) {
    await askNextQuestion()
    return
  }

  isThinking.value = true
  await wait(1000)
  isThinking.value = false
  isScreeningFinished.value = true
  conversation.value.push({
    role: 'assistant',
    text: 'Screening questions completed. You can upload another CV to restart the round.'
  })
}

onBeforeUnmount(() => {
  if (previewUrl.value) {
    URL.revokeObjectURL(previewUrl.value)
  }
})
</script>

<template>
  <section class="panel cvs-page">
    <h2>CV Screen</h2>
    <p class="sub">Upload a CV first, then answer AI questions generated step by step.</p>

    <div class="cvs-layout">
      <article class="cvs-top-card">
        <header class="cvs-card-head">
          <h3>CV Upload and Preview</h3>
          <button type="button" class="cvs-upload-btn" @click="openUploadDialog">Upload Resume</button>
          <input ref="fileInputRef" class="cvs-hidden-input" type="file" accept=".pdf,.doc,.docx,.txt,.png,.jpg,.jpeg" @change="onFileChange" />
        </header>

        <p class="cvs-file-name">
          {{ uploadedFileName ? `Current File: ${uploadedFileName}` : 'No file uploaded yet.' }}
        </p>

        <div class="cvs-preview-wrap">
          <iframe v-if="previewUrl" :src="previewUrl" title="CV Preview" />
          <div v-else class="cvs-preview-placeholder">Upload a CV to see preview here.</div>
        </div>
      </article>

      <article class="cvs-bottom-card">
        <header class="cvs-card-head">
          <h3>Q and A Screening</h3>
          <span class="cvs-status" :class="{ active: isThinking }">
            <template v-if="isThinking">
              Thinking
              <span class="cvs-thinking-dots" aria-hidden="true"><i>.</i><i>.</i><i>.</i></span>
            </template>
            <template v-else>Ready</template>
          </span>
        </header>

        <div class="cvs-chat-list">
          <div v-if="!isUploaded" class="cvs-chat-empty">
            Upload a CV first. The system will think for a moment and then ask the first question.
          </div>

          <div v-for="(item, index) in conversation" :key="`${item.role}-${index}`" class="cvs-chat-item" :class="item.role">
            <strong>{{ item.role === 'assistant' ? 'AI' : 'You' }}</strong>
            <p>{{ item.text }}</p>
          </div>

          <div v-if="isThinking" class="cvs-chat-item assistant">
            <strong>AI</strong>
            <p>
              Analyzing your CV and previous answer
              <span class="cvs-thinking-dots" aria-hidden="true"><i>.</i><i>.</i><i>.</i></span>
            </p>
          </div>
        </div>

        <form v-if="!isScreeningFinished" class="cvs-answer-form" @submit.prevent="submitAnswer">
          <input
            v-model="answerInput"
            type="text"
            placeholder="Type your answer..."
            :disabled="!isUploaded || isThinking"
          />
          <button type="submit" :disabled="!isUploaded || isThinking">Send</button>
        </form>
      </article>
    </div>
  </section>
</template>
