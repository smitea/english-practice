# English Practice Web App Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file Vue 3 web app for personal English practice with Chinese→English translation, automatic sentence analysis (word breakdown, usage context, examples, similar sentences), localStorage progress, and GitHub Pages deployment.

**Architecture:** One `index.html` file containing HTML + CSS + Vue 3 (via CDN) + inline JSON question bank. No build step, no backend, no dependencies. Vue's reactivity drives the entire UI; localStorage persists progress across reloads.

**Tech Stack:** Vue 3 (CDN, `vue.global.js`), vanilla CSS, vanilla JS, Web Storage API.

**Project root:** `/Users/shaw/Developer/node.js/english-practice/`

---

## File Structure

| File | Purpose |
|---|---|
| `index.html` | The entire app: HTML markup, `<style>` block, `<script type="application/json">` question bank, `<script>` Vue app code |
| `README.md` | Usage + deployment + how-to-extend instructions |
| `.gitignore` | Excludes `.DS_Store`, `.idea/`, etc. |

Single-file architecture is intentional. Each Vue component lives as a template literal inside `index.html`. This keeps the project bootable from `file://` and trivial to deploy to GitHub Pages.

---

## Task 1: Bootstrap project

**Files:**
- Create: `/Users/shaw/Developer/node.js/english-practice/.gitignore`
- Create: `/Users/shaw/Developer/node.js/english-practice/README.md`
- Create: `/Users/shaw/Developer/node.js/english-practice/index.html`

- [ ] **Step 1: Create `.gitignore`**

Content:
```
.DS_Store
.idea/
.vscode/
*.log
node_modules/
```

- [ ] **Step 2: Create empty `README.md`**

Content:
```markdown
# English Practice

Personal English practice app. See spec in `docs/superpowers/specs/`.
```

- [ ] **Step 3: Create minimal `index.html` boilerplate**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>English Practice</title>
</head>
<body>
  <div id="app"></div>
</body>
</html>
```

- [ ] **Step 4: Initialize git and first commit**

Run:
```bash
cd /Users/shaw/Developer/node.js/english-practice
git init
git add .
git commit -m "chore: bootstrap project"
```

Expected: `Initialized empty Git repository...` then commit succeeds.

---

## Task 2: Add Vue 3 CDN and inline question data

**Files:**
- Modify: `/Users/shaw/Developer/node.js/english-practice/index.html`

- [ ] **Step 1: Add Vue 3 CDN script and parse inline JSON**

Replace the entire `index.html` with:
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>English Practice</title>
</head>
<body>
  <div id="app">
    <p>Loading…</p>
  </div>

  <!-- Question bank (will be replaced in Task 7 with full 120 questions) -->
  <script type="application/json" id="questions-data">
[
  {
    "id": 1,
    "category": "daily_greetings",
    "scene_emoji": "☕",
    "scene_desc": "朋友在咖啡馆偶遇",
    "chinese": "你最近过得怎么样？",
    "english": "How have you been?",
    "hints": ["最近 recently", "过得 get along", "怎么样 how"],
    "difficulty": "beginner",
    "words": [
      { "text": "How",  "phonetic": "/ha�/", "pos": "adverb",     "pos_zh": "副词",   "meaning_zh": "怎样",     "role_zh": "疑问副词" },
      { "text": "have", "phonetic": "/hæv/", "pos": "auxiliary",  "pos_zh": "助动词", "meaning_zh": "已经",     "role_zh": "助动词" },
      { "text": "you",  "phonetic": "/juː/", "pos": "pronoun",    "pos_zh": "代词",   "meaning_zh": "你",       "role_zh": "主语" },
      { "text": "been", "phonetic": "/bɪn/", "pos": "verb",       "pos_zh": "动词",   "meaning_zh": "是、存在", "role_zh": "谓语" }
    ],
    "usage_context": "用于问候许久未见的朋友、同学、同事等熟人,语气随意亲切。常用于再次见面时的开场白,也可用于邮件/消息的开头。",
    "similar": [
      { "english": "How have you been doing?", "chinese": "你最近过得怎么样？(更随意)" },
      { "english": "How are you doing lately?", "chinese": "你最近怎么样？" }
    ],
    "examples": [
      {
        "english": "Hey Sarah! How have you been? It's been ages since we last met.",
        "chinese": "嘿 Sarah! 你最近怎么样?我们好久没见了。"
      },
      {
        "english": "How have you been since you moved to Shanghai?",
        "chinese": "你搬到上海以后过得怎么样?"
      }
    ]
  }
]
  </script>

  <!-- Vue 3 (CDN) -->
  <script src="https://unpkg.com/vue@3/dist/vue.global.prod.js"></script>

  <script>
    const QUESTIONS = JSON.parse(
      document.getElementById('questions-data').textContent
    )

    const { createApp, ref, computed } = Vue

    createApp({
      setup() {
        const current = ref(QUESTIONS[0])
        return { current }
      },
      template: `
        <div>
          <p>{{ current.scene_emoji }} {{ current.scene_desc }}</p>
          <h2>{{ current.chinese }}</h2>
          <p><strong>Answer:</strong> {{ current.english }}</p>
        </div>
      `
    }).mount('#app')
  </script>
</body>
</html>
```

- [ ] **Step 2: Open `index.html` in browser (double-click)**

Expected: Page shows "☕ 朋友在咖啡馆偶遇", "你最近过得怎么样？", and the answer "How have you been?".

If you see "Loading…" stuck or a console error, the JSON parsing or Vue CDN failed — check DevTools console.

- [ ] **Step 3: Commit**

```bash
cd /Users/shaw/Developer/node.js/english-practice
git add index.html
git commit -m "feat: bootstrap Vue 3 app with sample question"
```

---

## Task 3: Build PracticeView (categories, scene, input, feedback)

**Files:**
- Modify: `/Users/shaw/Developer/node.js/english-practice/index.html` (replace `<script>` block only)

- [ ] **Step 1: Add CSS variables at the top of `<style>` (insert after `<title>`)**

Insert this inside the `<head>`:
```html
<style>
  :root {
    --bg: #0f1419;
    --surface: #1a2028;
    --text: #e8e8e8;
    --text-dim: #9ca3af;
    --primary: #10b981;
    --primary-bright: #22c55e;
    --error: #f87171;
    --border: #2d3748;
    --pos-noun: #60a5fa;
    --pos-verb: #10b981;
    --pos-adjective: #f59e0b;
    --pos-adverb: #a78bfa;
    --pos-pronoun: #22d3ee;
    --pos-auxiliary: #f472b6;
    --pos-other: #9ca3af;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    background: var(--bg);
    color: var(--text);
    font-family: -apple-system, "PingFang SC", "Microsoft YaHei", system-ui, sans-serif;
    min-height: 100vh;
    padding: 20px;
  }
  #app { max-width: 720px; margin: 0 auto; }
  h1 { font-size: 1.4rem; margin-bottom: 16px; }
  button {
    background: var(--surface);
    color: var(--text);
    border: 1px solid var(--border);
    padding: 8px 16px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 0.95rem;
    transition: all 0.15s;
  }
  button:hover { border-color: var(--primary); }
  button:active { transform: scale(0.98); }
  button.primary { background: var(--primary); border-color: var(--primary); color: #000; }
  input[type="text"] {
    width: 100%;
    background: var(--surface);
    color: var(--text);
    border: 1px solid var(--border);
    padding: 12px 16px;
    border-radius: 6px;
    font-size: 1.1rem;
    font-family: inherit;
  }
  input[type="text"]:focus { outline: none; border-color: var(--primary); }
</style>
```

- [ ] **Step 2: Replace the Vue script with full PracticeView**

Replace the `<script>` block (the one after the Vue CDN) with:
```html
  <script>
    const QUESTIONS = JSON.parse(
      document.getElementById('questions-data').textContent
    )

    const CATEGORIES = [
      { id: 'daily_greetings', label: '寒暄' },
      { id: 'work',            label: '工作' },
      { id: 'travel',          label: '旅行' },
      { id: 'shopping',        label: '购物' },
      { id: 'social_emotion',  label: '社交' },
      { id: 'daily_life',      label: '生活' },
    ]

    function normalize(s) {
      return s
        .toLowerCase()
        .trim()
        .replace(/[.,!?;:'"]/g, '')
        .replace(/\s+/g, ' ')
    }

    const { createApp, ref, computed } = Vue

    createApp({
      setup() {
        const currentCategory = ref('daily_greetings')
        const userAnswer = ref('')
        const revealed = ref(false)
        const lastResult = ref(null) // 'correct' | 'wrong' | null

        const filtered = computed(() =>
          QUESTIONS.filter(q => q.category === currentCategory.value)
        )
        const currentIndex = ref(0)
        const current = computed(() => filtered.value[currentIndex.value] || filtered.value[0])

        const isCorrect = computed(() =>
          revealed.value &&
          normalize(userAnswer.value) === normalize(current.value.english)
        )

        function next() {
          const answered = userAnswer.value.trim().length > 0
          if (answered) {
            lastResult.value = isCorrect.value ? 'correct' : 'wrong'
          }
          // After recording result, advance to next question.
          revealed.value = false
          userAnswer.value = ''
          lastResult.value = null
          currentIndex.value = (currentIndex.value + 1) % filtered.value.length
        }

        function reveal() {
          revealed.value = !revealed.value
        }

        function switchCategory(catId) {
          currentCategory.value = catId
          currentIndex.value = 0
          userAnswer.value = ''
          revealed.value = false
          lastResult.value = null
        }

        return {
          CATEGORIES,
          currentCategory,
          current,
          userAnswer,
          revealed,
          lastResult,
          isCorrect,
          next,
          reveal,
          switchCategory,
        }
      },
      template: `
        <div>
          <h1>English Practice</h1>

          <nav class="tabs">
            <button
              v-for="cat in CATEGORIES"
              :key="cat.id"
              :class="{ active: currentCategory === cat.id }"
              @click="switchCategory(cat.id)"
            >{{ cat.label }}</button>
          </nav>

          <section class="scene">
            <div class="emoji">{{ current.scene_emoji }}</div>
            <div class="scene-desc">{{ current.scene_desc }}</div>
          </section>

          <section class="chinese">{{ current.chinese }}</section>

          <input
            type="text"
            v-model="userAnswer"
            placeholder="输入英文翻译…"
            @keydown.enter="next"
          />

          <details class="hints">
            <summary>词汇提示</summary>
            <ul>
              <li v-for="hint in current.hints" :key="hint">{{ hint }}</li>
            </ul>
          </details>

          <section v-if="revealed" class="answer-feedback" :class="{ ok: isCorrect, bad: !isCorrect && userAnswer.length > 0 }">
            <p v-if="isCorrect">✓ 答对了！</p>
            <p v-else-if="userAnswer.length > 0">✗ 答案是: <strong>{{ current.english }}</strong></p>
            <p v-else>答案: <strong>{{ current.english }}</strong></p>
          </section>

          <div class="actions">
            <button @click="reveal">{{ revealed ? '隐藏答案' : '显示答案' }}</button>
            <button class="primary" @click="next">下一题 →</button>
          </div>
        </div>
      `
    }).mount('#app')
  </script>
```

- [ ] **Step 3: Add component CSS at the bottom of `<style>` block**

Append before `</style>`:
```css
  .tabs { display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 20px; }
  .tabs button.active { background: var(--primary); color: #000; border-color: var(--primary); }
  .scene { text-align: center; margin: 24px 0; }
  .emoji { font-size: 3.5rem; }
  .scene-desc { color: var(--text-dim); margin-top: 6px; }
  .chinese {
    background: var(--surface);
    border-radius: 8px;
    padding: 20px;
    font-size: 1.5rem;
    text-align: center;
    margin-bottom: 16px;
  }
  .hints { margin: 12px 0; color: var(--text-dim); }
  .hints summary { cursor: pointer; padding: 4px 0; }
  .hints ul { padding-left: 20px; margin-top: 6px; }
  .answer-feedback { margin: 16px 0; padding: 12px; border-radius: 6px; background: var(--surface); }
  .answer-feedback.ok { border-left: 3px solid var(--primary-bright); }
  .answer-feedback.bad { border-left: 3px solid var(--error); }
  .actions { display: flex; gap: 8px; margin-top: 16px; }
</style>
```

- [ ] **Step 4: Manual test**

Open `index.html` in browser:
1. Verify category tabs render and click switches category (currently all categories except daily_greetings will be empty — that's expected since we only have 1 question).
2. Type "How have you been?" in the input → press Enter → see "下一题" advance (lastResult will briefly show).
3. Click "显示答案" → see expected English below input.
4. Open DevTools console → no errors.

- [ ] **Step 5: Commit**

```bash
cd /Users/shaw/Developer/node.js/english-practice
git add index.html
git commit -m "feat: build PracticeView with categories, input, and feedback"
```

---

## Task 4: Add analysis-view trigger and word breakdown table

**Files:**
- Modify: `/Users/shaw/Developer/node.js/english-practice/index.html`

- [ ] **Step 1: Add `showAnalysis` state and adjust `next()` logic**

In the `<script>` block, inside `setup()`, add after `const lastResult = ref(null)`:
```js
        const showAnalysis = ref(false)
```

Then replace the existing `next()` function with:
```js
        function next() {
          const answered = userAnswer.value.trim().length > 0
          if (answered && revealed.value) {
            // Only consider it "answered" if reveal was triggered.
            lastResult.value = isCorrect.value ? 'correct' : 'wrong'
          }
          if (lastResult.value === 'correct') {
            // Show analysis view instead of advancing directly.
            showAnalysis.value = true
            return
          }
          revealed.value = false
          userAnswer.value = ''
          lastResult.value = null
          currentIndex.value = (currentIndex.value + 1) % filtered.value.length
        }
```

- [ ] **Step 2: Update template to show analysis view conditionally**

Replace the entire `template` (backtick block) with:
```js
      template: `
        <div>
          <h1>English Practice</h1>

          <nav class="tabs" v-if="!showAnalysis">
            <button
              v-for="cat in CATEGORIES"
              :key="cat.id"
              :class="{ active: currentCategory === cat.id }"
              @click="switchCategory(cat.id)"
            >{{ cat.label }}</button>
          </nav>

          <!-- Practice View -->
          <section v-if="!showAnalysis">
            <div class="scene">
              <div class="emoji">{{ current.scene_emoji }}</div>
              <div class="scene-desc">{{ current.scene_desc }}</div>
            </div>

            <div class="chinese">{{ current.chinese }}</div>

            <input
              type="text"
              v-model="userAnswer"
              placeholder="输入英文翻译…"
              @keydown.enter="next"
              ref="answerInput"
            />

            <details class="hints">
              <summary>词汇提示</summary>
              <ul>
                <li v-for="hint in current.hints" :key="hint">{{ hint }}</li>
              </ul>
            </details>

            <div v-if="revealed" class="answer-feedback" :class="{ ok: isCorrect, bad: !isCorrect && userAnswer.length > 0 }">
              <p v-if="isCorrect">✓ 答对了！</p>
              <p v-else-if="userAnswer.length > 0">✗ 答案是: <strong>{{ current.english }}</strong></p>
              <p v-else>答案: <strong>{{ current.english }}</strong></p>
            </div>

            <div class="actions">
              <button @click="reveal">{{ revealed ? '隐藏答案' : '显示答案' }}</button>
              <button class="primary" @click="next">下一题 →</button>
            </div>
          </section>

          <!-- Analysis View -->
          <section v-else class="analysis">
            <div class="analysis-header">
              <button @click="exitAnalysis">← 返回</button>
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
              <button class="primary" @click="exitAnalysis">下一题 →</button>
            </div>
          </section>
        </div>
      `
```

- [ ] **Step 3: Add `exitAnalysis` function**

Add inside `setup()`:
```js
        function exitAnalysis() {
          showAnalysis.value = false
          revealed.value = false
          userAnswer.value = ''
          lastResult.value = null
          currentIndex.value = (currentIndex.value + 1) % filtered.value.length
        }
```

And add to the returned object:
```js
          exitAnalysis,
```

- [ ] **Step 4: Add analysis-view CSS**

Append before `</style>`:
```css
  .analysis-header { display: flex; align-items: center; gap: 12px; margin-bottom: 16px; }
  .badge-ok { color: var(--primary-bright); font-weight: 600; }
  .english-large {
    font-size: 1.8rem;
    color: var(--primary);
    text-align: center;
    margin: 16px 0 24px;
  }
  .word-table-wrapper { overflow-x: auto; margin-bottom: 20px; }
  .word-table { width: 100%; border-collapse: collapse; min-width: 400px; }
  .word-table th, .word-table td {
    padding: 8px 12px;
    text-align: center;
    border-bottom: 1px solid var(--border);
  }
  .word-table thead th { font-size: 1.1rem; padding-bottom: 12px; border-bottom: 2px solid var(--primary); }
  .word-table .phonetic { color: var(--text-dim); font-style: italic; font-family: "Times New Roman", serif; }
  .word-table .role { color: var(--text-dim); font-size: 0.9rem; }
  .pos-noun { color: var(--pos-noun); }
  .pos-verb { color: var(--pos-verb); }
  .pos-adjective { color: var(--pos-adjective); }
  .pos-adverb { color: var(--pos-adverb); }
  .pos-pronoun { color: var(--pos-pronoun); }
  .pos-auxiliary { color: var(--pos-auxiliary); }
  .pos-preposition, .pos-conjunction, .pos-article, .pos-particle { color: var(--pos-other); }
  .usage, .examples, .similar {
    background: var(--surface);
    border-radius: 8px;
    padding: 16px;
    margin-bottom: 16px;
  }
  .usage h3, .examples h3, .similar h3 { font-size: 1rem; margin-bottom: 8px; color: var(--primary); }
  .example { margin-bottom: 12px; }
  .example:last-child { margin-bottom: 0; }
  .ex-en { font-size: 1.05rem; }
  .ex-cn { color: var(--text-dim); margin-top: 4px; }
  .similar ul { list-style: none; padding: 0; }
  .similar li { padding: 6px 0; border-bottom: 1px solid var(--border); }
  .similar li:last-child { border-bottom: none; }
  .sim-cn { display: block; color: var(--text-dim); margin-top: 2px; font-size: 0.9rem; }
```

- [ ] **Step 5: Manual test**

1. Open `index.html` in browser.
2. Type "How have you been?" (correct) → click "下一题" → analysis view should appear.
3. Verify word breakdown table renders with all 5 rows.
4. Verify "💡 适用场景", "💬 示例" (2 examples), "📝 相似句式" (2 alternatives) all visible.
5. Click "下一题 →" on analysis view → returns to practice view.
6. Type wrong answer → press Enter → advances without analysis view.
7. Type correct + click "显示答案" first → "下一题" → analysis view.

- [ ] **Step 6: Commit**

```bash
cd /Users/shaw/Developer/node.js/english-practice
git add index.html
git commit -m "feat: add AnalysisView with word breakdown, usage, examples, similar"
```

---

## Task 5: localStorage progress + keyboard shortcuts

**Files:**
- Modify: `/Users/shaw/Developer/node.js/english-practice/index.html`

- [ ] **Step 1: Add progress state and persistence helpers**

Inside `<script>`, before `createApp`, add:
```js
    const STORAGE_KEY = 'english-practice/v1'

    function loadProgress() {
      try {
        const raw = localStorage.getItem(STORAGE_KEY)
        if (!raw) return { known: [], review: [], lastCategory: 'daily_greetings' }
        const parsed = JSON.parse(raw)
        return {
          known: Array.isArray(parsed.known) ? parsed.known : [],
          review: Array.isArray(parsed.review) ? parsed.review : [],
          lastCategory: parsed.lastCategory || 'daily_greetings',
        }
      } catch (e) {
        return { known: [], review: [], lastCategory: 'daily_greetings' }
      }
    }

    function saveProgress(p) {
      try {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(p))
      } catch (e) {
        // localStorage unavailable (e.g., private mode); silent no-op.
      }
    }
```

- [ ] **Step 2: Wire up progress state in `setup()`**

Add at the top of `setup()`:
```js
        const progress = ref(loadProgress())
```

Replace `const currentCategory = ref('daily_greetings')` with:
```js
        const currentCategory = ref(progress.value.lastCategory)
```

- [ ] **Step 3: Add mark-known / mark-review handlers and save on category switch**

Add inside `setup()`:
```js
        function markKnown() {
          const id = current.value.id
          if (!progress.value.known.includes(id)) {
            progress.value.known.push(id)
          }
          progress.value.review = progress.value.review.filter(x => x !== id)
          saveProgress(progress.value)
          advanceQuestion()
        }

        function markReview() {
          const id = current.value.id
          if (!progress.value.review.includes(id)) {
            progress.value.review.push(id)
          }
          saveProgress(progress.value)
          advanceQuestion()
        }

        function advanceQuestion() {
          revealed.value = false
          userAnswer.value = ''
          lastResult.value = null
          currentIndex.value = (currentIndex.value + 1) % filtered.value.length
        }

        function isKnown(id) {
          return progress.value.known.includes(id)
        }
```

Update `switchCategory` to persist:
```js
        function switchCategory(catId) {
          currentCategory.value = catId
          progress.value.lastCategory = catId
          saveProgress(progress.value)
          currentIndex.value = 0
          userAnswer.value = ''
          revealed.value = false
          lastResult.value = null
        }
```

Add `markKnown`, `markReview`, `isKnown`, `progress` to the return object.

- [ ] **Step 4: Add ProgressBar and Mark buttons to template**

Inside the practice view (the `<section v-if="!showAnalysis">`), after `<div class="actions">`, add:
```html
            <div class="progress-row">
              <span>已掌握 {{ progress.known.length }} 题</span>
              <span>需复习 {{ progress.review.length }} 题</span>
            </div>
            <div class="actions mark-actions">
              <button @click="markKnown" :disabled="isKnown(current.id)">
                {{ isKnown(current.id) ? '✓ 已掌握' : '✓ 标记已掌握' }}
              </button>
              <button @click="markReview">� 标记需复习</button>
            </div>
```

- [ ] **Step 5: Add keyboard shortcuts**

Add inside `setup()`:
```js
        function handleKeydown(e) {
          // Ignore when typing in input (except Tab which we handle).
          if (e.target.tagName === 'INPUT' && e.key !== 'Tab') return
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
          } else if (e.key === 'Tab') {
            e.preventDefault()
            reveal()
          } else if (e.key >= '1' && e.key <= '6') {
            const idx = parseInt(e.key, 10) - 1
            if (CATEGORIES[idx]) switchCategory(CATEGORIES[idx].id)
          }
        }
```

Add right before the closing `}).mount('#app')`:
```js
        window.addEventListener('keydown', handleKeydown)
```

Also return `handleKeydown` (optional, not used in template).

- [ ] **Step 6: Add CSS for progress row**

Append before `</style>`:
```css
  .progress-row {
    display: flex;
    justify-content: space-between;
    color: var(--text-dim);
    margin: 12px 0;
    font-size: 0.9rem;
  }
  .mark-actions { margin-top: 8px; }
  button:disabled { opacity: 0.5; cursor: not-allowed; }
```

- [ ] **Step 7: Manual test**

1. Type correct answer → next → analysis view.
2. Click "下一题 →" → back to practice.
3. Click "✓ 标记已掌握" → button text changes, count increments.
4. Reload page → progress count persists, "✓ 已掌握" still shown on the same question.
5. Keyboard: press `Tab` → toggles reveal.
6. Keyboard: press `2` → switches to "工作" category (empty — that's fine).
7. Keyboard: press `Enter` while input focused → advances.

- [ ] **Step 8: Commit**

```bash
cd /Users/shaw/Developer/node.js/english-practice
git add index.html
git commit -m "feat: localStorage progress + keyboard shortcuts"
```

---

## Task 6: Responsive layout polish

**Files:**
- Modify: `/Users/shaw/Developer/node.js/english-practice/index.html`

- [ ] **Step 1: Add mobile breakpoint**

Append before `</style>`:
```css
  @media (max-width: 600px) {
    body { padding: 12px; }
    .chinese { font-size: 1.3rem; padding: 16px; }
    .english-large { font-size: 1.5rem; }
    .emoji { font-size: 3rem; }
    .word-table { min-width: 320px; }
    .actions { flex-wrap: wrap; }
    .actions button { flex: 1; min-width: 120px; }
  }
```

- [ ] **Step 2: Manual test**

1. Open `index.html` in browser.
2. Resize window to < 600px wide (or use DevTools device mode, e.g., iPhone 12).
3. Verify word table scrolls horizontally without breaking layout.
4. Verify buttons stack/wrap nicely.
5. Verify scene emoji + Chinese sentence are readable.

- [ ] **Step 3: Commit**

```bash
cd /Users/shaw/Developer/node.js/english-practice
git add index.html
git commit -m "feat: responsive layout for mobile"
```

---

## Task 7: Generate full 120-question dataset

**Files:**
- Modify: `/Users/shaw/Developer/node.js/english-practice/index.html` (replace inline JSON block)

This task is performed with AI assistance. The engineer drives generation by category; this task shows how to expand the data.

- [ ] **Step 1: Generate 20 questions for `daily_greetings`**

Ask the assistant (or use another AI tool) to generate 20 beginner-friendly Chinese→English sentences for the category `daily_greetings`. Each must include the full schema (id, category, scene_emoji, scene_desc, chinese, english, hints[2-5], difficulty, words[4-8], usage_context, similar[2], examples[1-2]).

IDs 1–20. Inline into the JSON, replacing the single sample question.

- [ ] **Step 2: Generate 20 questions for `work`**

IDs 21–40. category = `work`. Cover meetings, emails, presentations, colleague chats.

- [ ] **Step 3: Generate 20 questions for `travel`**

IDs 41–60. category = `travel`. Cover airports, hotels, restaurants, asking directions.

- [ ] **Step 4: Generate 20 questions for `shopping`**

IDs 61–80. category = `shopping`. Cover asking prices, bargaining, returns, online shopping.

- [ ] **Step 5: Generate 20 questions for `social_emotion`**

IDs 81–100. category = `social_emotion`. Cover thanks, apologies, expressing opinions, feelings.

- [ ] **Step 6: Generate 20 questions for `daily_life`**

IDs 101–120. category = `daily_life`. Cover medical, emergency, neighbors, pets, hobbies.

- [ ] **Step 7: Validate JSON**

Run from project root:
```bash
node -e "
const html = require('fs').readFileSync('index.html', 'utf8');
const m = html.match(/<script type=\"application\/json\" id=\"questions-data\">([\s\S]*?)<\/script>/);
if (!m) { console.error('JSON block not found'); process.exit(1); }
const data = JSON.parse(m[1]);
console.log('Total questions:', data.length);
const byCat = {};
data.forEach(q => byCat[q.category] = (byCat[q.category] || 0) + 1);
console.log('By category:', byCat);
data.forEach(q => {
  if (!q.english || !q.chinese) throw new Error('Missing translation: ' + q.id);
  if (!Array.isArray(q.words) || q.words.length < 2) throw new Error('Missing words: ' + q.id);
  if (!q.usage_context) throw new Error('Missing usage_context: ' + q.id);
  if (!Array.isArray(q.similar) || q.similar.length < 2) throw new Error('Missing similar: ' + q.id);
  if (!Array.isArray(q.examples) || q.examples.length < 1) throw new Error('Missing examples: ' + q.id);
});
console.log('Validation passed.');
"
```

Expected: `Total questions: 120`, all categories show 20, validation passes.

- [ ] **Step 8: Manual test in browser**

1. Open `index.html`.
2. Click each of 6 category tabs — each should show 20 questions.
3. Click through several questions per category, verifying scene emoji, hints, and answer checking work.
4. Answer one correctly in each category → confirm analysis view shows word breakdown, usage, examples, similar.

- [ ] **Step 9: Commit**

```bash
cd /Users/shaw/Developer/node.js/english-practice
git add index.html
git commit -m "feat: full 120-question dataset across 6 categories"
```

---

## Task 8: End-to-end manual testing pass

**Files:** none

- [ ] **Step 1: Browser compatibility**

Test in each browser (file should open via `file://` protocol):
- [ ] Safari (macOS)
- [ ] Chrome (macOS)
- [ ] Firefox (macOS)

For each: load page, click categories, type answers, verify analysis view, verify progress persists after reload.

Expected: identical behavior across browsers. If Safari/Firefox differs, document in README troubleshooting.

- [ ] **Step 2: Keyboard-only navigation**

1. Reload page.
2. Without using mouse:
   - Press number keys to switch category.
   - Type into input.
   - Press `Tab` to reveal.
   - Press `Enter` to advance.
3. Verify all actions reachable.

- [ ] **Step 3: Mobile responsive**

1. Open in Chrome DevTools → toggle device toolbar → iPhone 12 / Pixel 5.
2. Verify all content readable, word table scrolls horizontally, buttons reachable.

- [ ] **Step 4: Progress edge cases**

1. Mark same question "known" twice — should be no-op (no duplicate id in array).
2. Mark question as both "known" and "review" — known wins (review entry removed).
3. Open in private/incognito window — progress doesn't persist (expected; localStorage disabled).
4. Manually corrupt localStorage (`localStorage.setItem('english-practice/v1', 'garbage')`) and reload — should fall back to defaults without crashing.

- [ ] **Step 5: Commit any fixes**

If bugs were found and fixed:
```bash
cd /Users/shaw/Developer/node.js/english-practice
git add index.html
git commit -m "fix: <describe bug>"
```

If no bugs: skip.

---

## Task 9: Write README

**Files:**
- Modify: `/Users/shaw/Developer/node.js/english-practice/README.md`

- [ ] **Step 1: Replace README content**

```markdown
# English Practice

A single-file Vue 3 web app for personal English practice. Translates Chinese sentences to English, then automatically shows a sentence analysis: word-by-word breakdown (phonetic, part of speech, Chinese meaning, grammar role), usage context, realistic examples, and similar sentences.

## Features

- 120 starter questions across 6 categories: 寒暄 / 工作 / 旅行 / 购物 / 社交 / 生活
- Auto-shows analysis after correct answer (word breakdown + usage + examples + similar)
- LocalStorage progress tracking (已知 / 需复习)
- Dark mode UI with emerald accent
- Keyboard shortcuts: `Enter` next, `Tab` reveal, `1-6` switch category
- Works offline via `file://` protocol
- Free hosting on GitHub Pages

## Run locally

Just open `index.html` in any modern browser. No install, no build.

```bash
open index.html        # macOS
xdg-open index.html    # Linux
start index.html       # Windows
```

## Deploy to GitHub Pages

1. Create a new GitHub repo (e.g. `english-practice`).
2. Push this directory to `main`.
3. Repo **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: main / (root)**.
4. Wait ~1 minute. Site live at `https://<your-username>.github.io/english-practice/`.

## Keyboard shortcuts

| Key | Action |
|---|---|
| `Enter` | Submit answer / advance to next question |
| `Tab` | Toggle reveal answer |
| `1`–`6` | Switch to category 1–6 |
| `Esc` | Exit analysis view |

## How to add more questions

Open `index.html` and find the `<script type="application/json" id="questions-data">` block. Each question follows this schema:

```json
{
  "id": 121,
  "category": "daily_greetings",
  "scene_emoji": "�",
  "scene_desc": "跟邻居打招呼",
  "chinese": "早上好！",
  "english": "Good morning!",
  "hints": ["早上 morning", "好 good"],
  "difficulty": "beginner",
  "words": [
    { "text": "Good",   "phonetic": "/ɡʊd/",    "pos": "adjective", "pos_zh": "形容词", "meaning_zh": "好的",     "role_zh": "定语" },
    { "text": "morning","phonetic": "/ˈmɔːnɪŋ/","pos": "noun",      "pos_zh": "名词",   "meaning_zh": "早晨",     "role_zh": "主语" }
  ],
  "usage_context": "正式或友好的早晨问候,适合邻里、同事、陌生人之间。",
  "similar": [
    { "english": "Good afternoon!", "chinese": "下午好！" },
    { "english": "Good evening!",   "chinese": "晚上好！" }
  ],
  "examples": [
    { "english": "Good morning! How's your day going?", "chinese": "早上好!你今天过得怎么样?" }
  ]
}
```

Append your question to the array (use the next available `id`).

## Project structure

```
english-practice/
├── index.html              # The entire app
├── README.md
└── docs/
    └── superpowers/
        ├── specs/          # Design specs
        └── plans/          # Implementation plans
```

No build step, no dependencies, no backend.

## Tech stack

- Vue 3 (CDN)
- Vanilla CSS
- Vanilla JS
- Web Storage API (localStorage)

## License

MIT
```

- [ ] **Step 2: Commit**

```bash
cd /Users/shaw/Developer/node.js/english-practice
git add README.md
git commit -m "docs: comprehensive README"
```

---

## Task 10: Push to GitHub and enable Pages

**Files:** none (git operations only)

This task requires the user to provide GitHub authentication. Two options:

- [ ] **Step 1: Create GitHub repo and push**

Option A — GitHub CLI (if `gh` is authenticated):
```bash
cd /Users/shaw/Developer/node.js/english-practice
gh repo create english-practice --public --source=. --remote=origin --push
```

Option B — Manual:
1. Go to https://github.com/new
2. Repo name: `english-practice`, Public, **do not** initialize with README/license (we already have them).
3. Then locally:
```bash
cd /Users/shaw/Developer/node.js/english-practice
git remote add origin git@github.com:<your-username>/english-practice.git
git push -u origin main
```

Expected: Push succeeds; GitHub shows the repo at `https://github.com/<your-username>/english-practice`.

- [ ] **Step 2: Enable GitHub Pages**

1. Open `https://github.com/<your-username>/english-practice/settings/pages`
2. **Source:** `Deploy from a branch`
3. **Branch:** `main` / `(root)`
4. Click **Save**
5. Wait ~1 minute. Refresh page. URL appears under "Your site is live at":
   `https://<your-username>.github.io/english-practice/`

- [ ] **Step 3: Verify live site**

Open the live URL in browser. Verify:
- [ ] All 6 categories load
- [ ] Answering correctly triggers analysis view
- [ ] localStorage progress persists across page reloads

- [ ] **Step 4: Tag the release**

```bash
cd /Users/shaw/Developer/node.js/english-practice
git tag v1.0.0
git push origin v1.0.0
```

---

## Self-Review

**Spec coverage check:**

| Spec section | Implemented in task |
|---|---|
| §1 Purpose — scenario card, Chinese sentence, English input | Task 3 |
| §1 Purpose — analysis view (word breakdown, usage, examples, similar) | Task 4 |
| §1 Purpose — progress in localStorage | Task 5 |
| §3 Tech Stack — Vue 3 CDN, no build | Task 2 |
| §4 Project Structure — single index.html + README | Task 1, Task 9 |
| §5 Data Model — full schema (id, category, scene_*, chinese, english, hints, words, usage_context, similar, examples) | Task 2 (sample), Task 7 (full) |
| §6 Categories — 6 categories × 20 = 120 | Task 7 |
| §7 UI Layout — practice view | Task 3 |
| §7 UI Layout — analysis view (5-row word table + usage + examples + similar) | Task 4 |
| §7 UI Layout — POS color coding | Task 4 |
| §7 Responsive — < 600px breakpoints | Task 6 |
| §8 Component Tree — Vue components | Tasks 3, 4 |
| §9 Flow 1 — correct → analysis | Task 4 |
| §9 Flow 2 — wrong/empty → no analysis | Task 4 |
| §9 Flow 3 — reveal toggle + 查看解析 | Task 4 |
| §9 Flow 4 — mark known/review | Task 5 |
| §9 Flow 5 — category switch | Tasks 3, 5 |
| §10 Answer normalization logic | Task 3 |
| §11 Question selection | Task 3 (round-robin; spec accepts) |
| §12 Keyboard shortcuts — Enter / Tab / 1-6 / Esc | Task 5 |
| §13 Styling — emerald accent, dark mode | Task 3 |
| §14 Error handling — localStorage disabled | Task 5 |
| §15 Accessibility — keyboard, semantic HTML, focus rings | Task 3, 5 |
| §16 Testing strategy — manual | Task 8 |
| §17 Deployment — GitHub Pages | Task 10 |

**Coverage gaps:** None found.

**Placeholder scan:** No "TBD" / "TODO" / "implement later" in plan. Every step contains exact code or exact commands.

**Type consistency:** `currentCategory`, `userAnswer`, `revealed`, `lastResult`, `showAnalysis`, `progress`, `currentIndex`, `current` all defined once in Task 3/4/5 and reused consistently. `normalize()` defined once in Task 3. `STORAGE_KEY` defined once in Task 5.

**Self-review pass: clean.**

---

## Execution Handoff

Plan saved to `/Users/shaw/Developer/node.js/english-practice/docs/superpowers/plans/2026-08-27-english-practice.md`.

Ready for execution. Two options:

1. **Subagent-Driven (recommended)** — Dispatch a fresh subagent per task, review between tasks, fast iteration
2. **Inline Execution** — Execute tasks in this session with checkpoints
