<script setup>
import WordSlots from './WordSlots.vue'

const props = defineProps({
  current: Object,
  userAnswer: String,
  isCorrect: Boolean,
  revealed: Boolean,
  slotInputs: Array,
})

const emit = defineEmits(['next', 'submit'])

function onSubmit() {
  emit('next')
}
</script>

<template>
  <section>
    <div class="scene">
      <div class="emoji">{{ current.scene_emoji }}</div>
      <div class="scene-desc">{{ current.scene_desc }}</div>
    </div>

    <div class="chinese">{{ current.chinese }}</div>

    <WordSlots
      :current="current"
      :revealed="revealed"
      :slot-inputs="slotInputs"
      @submit="onSubmit"
    />

    <div v-if="revealed && !isCorrect && userAnswer.length > 0" class="answer-feedback bad">
      <p>✗ 答案是: <strong>{{ current.english }}</strong></p>
    </div>
  </section>
</template>