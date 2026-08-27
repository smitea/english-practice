<script setup>
const props = defineProps({
  current: Object,
})

const emit = defineEmits(['exit'])

function onExit() {
  emit('exit')
}
</script>

<template>
  <section class="analysis">
    <div class="analysis-header">
      <button @click="onExit">← 返回</button>
      <span class="badge-ok">✓ 答对了！</span>
    </div>

    <h2 class="english-large">{{ current.english }}</h2>

    <div class="word-table-wrapper">
      <table class="word-table">
        <thead>
          <tr>
            <th v-for="(w, i) in current.words" :key="i" :class="'pos-' + w.pos">{{ w.text }}</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td v-for="(w, i) in current.words" :key="'p' + i" class="phonetic">{{ w.phonetic }}</td>
          </tr>
          <tr>
            <td v-for="(w, i) in current.words" :key="'pos' + i" :class="'pos-' + w.pos">{{ w.pos_zh }}</td>
          </tr>
          <tr>
            <td v-for="(w, i) in current.words" :key="'m' + i">{{ w.meaning_zh }}</td>
          </tr>
          <tr>
            <td v-for="(w, i) in current.words" :key="'r' + i" class="role">{{ w.role_zh }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="usage">
      <h3>💡 适用场景</h3>
      <p>{{ current.usage_context }}</p>
    </div>

    <div class="examples">
      <h3>💬 示例</h3>
      <div v-for="(ex, i) in current.examples" :key="i" class="example">
        <p class="ex-en">{{ ex.english }}</p>
        <p class="ex-cn">{{ ex.chinese }}</p>
      </div>
    </div>

    <div class="similar">
      <h3>📝 相似句式</h3>
      <ul>
        <li v-for="(s, i) in current.similar" :key="i">
          <span class="sim-en">{{ s.english }}</span>
          <span class="sim-cn">{{ s.chinese }}</span>
        </li>
      </ul>
    </div>

    <div class="actions">
      <button class="primary" @click="onExit">下一题 →</button>
    </div>
  </section>
</template>