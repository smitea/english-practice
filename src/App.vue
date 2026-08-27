<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import questions from './data/questions.json'
import PracticeView from './components/PracticeView.vue'
import AnalysisView from './components/AnalysisView.vue'

const QUESTIONS = questions

const slotInputs = ref([])
const revealed = ref(false)
const showAnalysis = ref(false)
const currentIndex = ref(0)
const lastResult = ref(null)

const current = computed(() => QUESTIONS[currentIndex.value] || QUESTIONS[0])

const userAnswer = computed(() =>
  slotInputs.value.map(s => s.trim()).filter(Boolean).join(' ')
)

function normalize(s) {
  return s
    .toLowerCase()
    .trim()
    .replace(/[.,!?;:'"]/g, '')
    .replace(/\s+/g, ' ')
}

const isCorrect = computed(() =>
  userAnswer.value.trim().length > 0 &&
  normalize(userAnswer.value) === normalize(current.value.english)
)

function resetSlots() {
  const expected = current.value.english
    .replace(/[.,!?;:'"]/g, '')
    .split(/\s+/)
    .filter(Boolean)
  slotInputs.value = expected.map(() => '')
}

function focusFirstSlot() {
  setTimeout(() => {
    const first = document.querySelector('.slot-input')
    if (first) first.focus()
  }, 0)
}

function advance() {
  revealed.value = false
  lastResult.value = null
  currentIndex.value = (currentIndex.value + 1) % QUESTIONS.length
  resetSlots()
  focusFirstSlot()
}

function next() {
  if (userAnswer.value.trim().length === 0) {
    advance()
    return
  }
  if (isCorrect.value) {
    if (revealed.value) {
      advance()
    } else {
      showAnalysis.value = true
    }
    return
  }
  if (!revealed.value) {
    revealed.value = true
  }
}

function exitAnalysis() {
  showAnalysis.value = false
  revealed.value = false
  lastResult.value = null
  currentIndex.value = (currentIndex.value + 1) % QUESTIONS.length
  resetSlots()
  focusFirstSlot()
}

function handleKeydown(e) {
  if (e.target.tagName === 'INPUT') return
  if (showAnalysis.value) {
    if (e.key === 'Escape' || e.key === 'Enter') {
      e.preventDefault()
      exitAnalysis()
    }
    return
  }
  if (e.key === 'Enter') {
    e.preventDefault()
    next()
  }
}

onMounted(() => {
  resetSlots()
  focusFirstSlot()
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<template>
  <PracticeView
    v-if="!showAnalysis"
    :current="current"
    :user-answer="userAnswer"
    :is-correct="isCorrect"
    :revealed="revealed"
    :slot-inputs="slotInputs"
    @next="next"
  />
  <AnalysisView
    v-else
    :current="current"
    @exit="exitAnalysis"
  />
</template>