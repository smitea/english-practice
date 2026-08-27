<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  slotInputs: Array,
  current: Object,
  revealed: Boolean,
})

function normalize(s) {
  return s
    .toLowerCase()
    .trim()
    .replace(/[.,!?;:'"]/g, '')
    .replace(/\s+/g, ' ')
}

const wordSlots = computed(() => {
  const expected = props.current.english
    .replace(/[.,!?;:'"]/g, '')
    .split(/\s+/)
    .filter(Boolean)
  const typed = props.slotInputs
    .map(s => (s || '').replace(/[.,!?;:'"]/g, ''))
    .map(s => s.trim())
  return expected.map((word, i) => {
    const t = (typed[i] || '').split(/\s+/).filter(Boolean)[0] || ''
    return {
      expected: word,
      typed: t,
      isCorrect: !!t && normalize(t) === normalize(word),
    }
  })
})

function slotWidth(word) {
  return Math.max(56, word.length * 11 + 24) + 'px'
}

function onSlotInput(i, e) {
  const cleaned = e.target.value.replace(/[\s.,!?;:'"]/g, '')
  if (cleaned !== e.target.value) {
    props.slotInputs[i] = cleaned
  }
}

function focusNextSlot(i) {
  const inputs = document.querySelectorAll('.slot-input')
  const next = inputs[i + 1]
  if (next) next.focus()
}

function triggerShake(i) {
  const inputs = document.querySelectorAll('.slot-input')
  const input = inputs[i]
  if (!input) return
  input.classList.remove('shake')
  void input.offsetWidth
  input.classList.add('shake')
  setTimeout(() => input.classList.remove('shake'), 450)
}

function onSlotSpace(i) {
  const inputs = document.querySelectorAll('.slot-input')
  const v = props.slotInputs[i] || ''
  const expected = (props.current.english
    .replace(/[.,!?;:'"]/g, '')
    .split(/\s+/)[i]) || ''
  const isCorrect = !v || normalize(v) === normalize(expected)
  if (!isCorrect) {
    triggerShake(i)
    return
  }
  if (i + 1 < inputs.length) {
    inputs[i + 1].focus()
  } else {
    const enterEvt = new KeyboardEvent('keydown', { key: 'Enter' })
    inputs[i].dispatchEvent(enterEvt)
  }
}

function onSlotEnter() {
  // bubble up to parent via 'next' event
  emit('submit')
}

function onBackspace(i, e) {
  if (!props.slotInputs[i] && i > 0) {
    e.preventDefault()
    const inputs = document.querySelectorAll('.slot-input')
    const prev = inputs[i - 1]
    if (prev) {
      prev.focus()
      const len = prev.value.length
      prev.setSelectionRange(len, len)
    }
  }
}

const emit = defineEmits(['submit'])
</script>

<template>
  <div class="word-slots">
    <input
      v-for="(slot, i) in wordSlots"
      :key="i"
      type="text"
      v-model="props.slotInputs[i]"
      class="slot-input"
      :class="{
        'slot-correct': slot.typed && slot.isCorrect,
        'slot-incorrect': slot.typed && !slot.isCorrect,
        'slot-pending': !slot.typed,
        'slot-revealed': props.revealed
      }"
      :placeholder="props.revealed ? slot.expected : '_'"
      :style="{ width: slotWidth(slot.expected) }"
      @keydown.space.prevent="onSlotSpace(i)"
      @keydown.tab.prevent
      @keydown.enter.prevent="emit('submit')"
      @input="onSlotInput(i, $event)"
      @keydown.backspace="onBackspace(i, $event)"
    />
  </div>
</template>