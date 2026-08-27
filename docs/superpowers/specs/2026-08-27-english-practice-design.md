# English Practice Web App — Design Spec

**Date:** 2026-08-27
**Status:** Approved (pending user review of written spec)
**Author:** Brainstorming session

---

## 1. Purpose

A single-file web app for personal English practice, modeled after the interactive learning style seen in Chinese livestream English classes:

- Display a scenario card (emoji + short scene description)
- Show a Chinese sentence to translate
- User types the English translation
- After correct answer, show a **sentence analysis view**:
  - word-by-word breakdown (phonetic, part of speech, Chinese meaning, grammar role)
  - usage context (when/where this sentence is used)
  - 1–2 realistic dialogue examples with Chinese translations
  - similar sentences using the same pattern
- Reveal the answer; mark as known / needs-review
- Track progress per category in `localStorage`

**Scope:** Personal use only. No backend, no accounts, no network calls.

---

## 2. Goals & Non-Goals

### Goals
- Run by double-clicking `index.html` (works on `file://` protocol)
- Run from GitHub Pages (`https://<user>.github.io/english-practice/`)
- Zero build step, zero npm install required to use
- 120 starter questions across 6 categories, each with word-level analysis, usage context, 1–2 example dialogues, and 2 similar sentences
- Keyboard-friendly (Enter / Tab shortcuts)
- Mobile + desktop responsive

### Non-Goals (out of scope for v1)
- TTS / speech recognition (planned for v2 — listening & speaking modes)
- API-based question generation
- User accounts, sync, multi-device progress
- Spaced-repetition algorithm
- Statistics dashboard / streak tracking

---

## 3. Tech Stack

| Layer | Choice | Reason |
|---|---|---|
| Framework | Vue 3 (CDN, `vue.global.js`) | Simplicity, no build step, reactivity |
| Styling | Plain CSS (scoped via `<style>` block in HTML) | No preprocessor needed |
| Data | Inline JSON inside `<script type="application/json">` | Avoids `file://` CORS issues |
| Storage | `localStorage` | Built-in, no backend needed |
| Hosting | GitHub Pages (main branch, root) | Free, matches user's existing workflow |

**No package.json. No node_modules. No build pipeline.**

---

## 4. Project Structure

```
english-practice/
├── index.html      # Single file: HTML + CSS + Vue code + inline questions JSON
└── README.md       # Usage + deployment instructions
```

---

## 5. Data Model

### Question

```json
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
    { "text": "How",  "phonetic": "/haʊ/", "pos": "adverb",     "pos_zh": "副词",   "meaning_zh": "怎样",     "role_zh": "疑问副词" },
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
```

| Field | Type | Notes |
|---|---|---|
| `id` | number | Stable across edits |
| `category` | enum | See §6 |
| `scene_emoji` | string | Single emoji or short symbol |
| `scene_desc` | string | Chinese scene description (≤ 20 chars) |
| `chinese` | string | Source sentence (≤ 30 chars) |
| `english` | string | Canonical English translation (the answer) |
| `hints` | string[] | Chinese-English word pairs (2–5 items) |
| `difficulty` | enum | `beginner` \| `intermediate` |
| `words` | Word[] | Word-level breakdown (see below) — drives analysis view |
| `usage_context` | string | When/where this sentence is used (≤ 80 chars Chinese) |
| `similar` | SimilarSentence[] | 2 sentences using the same pattern |
| `examples` | Example[] | 1–2 realistic dialogue examples using this sentence |

#### Word

| Field | Type | Notes |
|---|---|---|
| `text` | string | The word/phrase as it appears in the sentence |
| `phonetic` | string | IPA transcription, e.g. `/haʊ/` |
| `pos` | enum | `noun` \| `verb` \| `adjective` \| `adverb` \| `pronoun` \| `auxiliary` \| `preposition` \| `conjunction` \| `article` \| `particle` |
| `pos_zh` | string | Chinese POS label, e.g. `副词` |
| `meaning_zh` | string | Chinese meaning in this context, ≤ 6 chars |
| `role_zh` | string | Chinese grammar role, e.g. `方式状语`, `主语`, `谓语`, `宾语`, `表语` |

#### SimilarSentence

| Field | Type | Notes |
|---|---|---|
| `english` | string | Alternative sentence using same pattern |
| `chinese` | string | Chinese translation, ≤ 30 chars |

#### Example

| Field | Type | Notes |
|---|---|---|
| `english` | string | Realistic dialogue example using the source sentence (≤ 60 chars) |
| `chinese` | string | Chinese translation, ≤ 50 chars |

### Progress (localStorage)

```json
{
  "english-practice/v1": {
    "known": [1, 7, 23],
    "review": [5, 12],
    "lastCategory": "daily_greetings"
  }
}
```

- `known`: questions marked as mastered by user
- `review`: questions marked as needs-review
- `lastCategory`: restore category filter on reload

---

## 6. Categories

20 questions per category = 120 total:

| ID | Category | Chinese Name |
|---|---|---|
| `daily_greetings` | Daily greetings | 日常寒暄 |
| `work` | Work & business | 工作职场 |
| `travel` | Travel | 旅行出行 |
| `shopping` | Shopping | 购物消费 |
| `social_emotion` | Social / emotional | 社交情感 |
| `daily_life` | Daily life | 生活场景 |

---

## 7. UI Layout

### Practice View

```
┌────────────────────────────────────────────┐
│  English Practice                  ☰      │  ← Header
├────────────────────────────────────────────┤
│ [寒暄] [工作] [旅行] [购物] [社交] [生活]   │  ← Category tabs
│ Progress: ████████░░ 80%                   │  ← Progress bar
├────────────────────────────────────────────┤
│              ☕                             │
│       朋友在咖啡馆偶遇                       │  ← Scene card
│                                            │
│   ┌────────────────────────────────┐       │
│   │    你最近过得怎么样？           │       │  ← Chinese sentence
│   └────────────────────────────────┘       │
│                                            │
│   ┌────────────────────────────────┐       │
│   │  How have you been_            │       │  ← English input
│   └────────────────────────────────┘       │
│                                            │
│   ▼ 词汇提示                                  │
│   最近 recently · 过得 get along · 怎么样 how │
├────────────────────────────────────────────┤
│  [显示答案]  [下一题]  [✓已掌握] [↻需复习] │  ← Actions
└────────────────────────────────────────────┘
```

### Analysis View (shown after correct answer, or via "查看解析" button)

```
┌────────────────────────────────────────────�
│  ← 返回          ✓ 答对了！                  │
├────────────────────────────────────────────┤
│  How have you been?                         │  ← English sentence (large)
├────────────────────────────────────────────┤
│  How    have   you   been                   │  ← Word row 1: text
│  /haʊ/ /hæv/ /juː/ /bɪn/                   │  ← Word row 2: phonetic
│  副词   助动词 代词  动词                    │  ← Word row 3: POS (color-coded)
│  怎样   已经   你    是/存在                │  ← Word row 4: Chinese meaning
│  状语   助动词 主语  谓语                    │  ← Word row 5: grammar role
├────────────────────────────────────────────�
│  💡 适用场景                                   │
│  用于问候许久未见的朋友、同学、同事等熟人,    │
│  语气随意亲切。常用于再次见面时的开场白,     │
│  也可用于邮件/消息的开头。                   │
├────────────────────────────────────────────┤
│  💬 示例                                       │
│  ┌────────────────────────────────────┐     │
│  │ Hey Sarah! How have you been? It's │     │
│  │ been ages since we last met.       │     │
│  │ 嘿 Sarah! 你最近怎么样?我们好久没见│     │
│  │ 了。                               │     │
│  └────────────────────────────────────┘     │
│  ┌────────────────────────────────────┐     │
│  │ How have you been since you moved  │     │
│  │ to Shanghai?                       │     │
│  │ 你搬到上海以后过得怎么样?          │     │
│  └────────────────────────────────────┘     │
├────────────────────────────────────────────┤
│  📝 相似句式                                  │
│  • How have you been doing?                 │
│    你最近过得怎么样？(更随意)               │
│  • How are you doing lately?                │
│    你最近怎么样？                            │
├────────────────────────────────────────────┤
│              [下一题 →]                     │
└────────────────────────────────────────────┘
```

Section order in analysis view (top → bottom):
1. English sentence (large)
2. Word breakdown table (5 rows × N words)
3. **💡 适用场景** (new — when to use this sentence)
4. **💬 示例** (new — 1–2 realistic dialogue examples with Chinese translation)
5. � 相似句式 (2 same-pattern alternatives)
6. 下一题 button

POS color coding (consistent across app):
- Noun: blue
- Verb: green (primary accent)
- Adjective: amber
- Adverb: purple
- Pronoun: cyan
- Auxiliary: pink
- Preposition / Conjunction: gray
- Article / Particle: gray

User can scroll horizontally on mobile to see all word columns.

### Responsive breakpoints
- `< 600px`: stack everything full-width, larger touch targets
- `≥ 600px`: max-width 720px centered

---

## 8. Component Tree (Vue)

```
App
├── PracticeView             (v-if="!showAnalysis")
│   ├── Header
│   ├── CategoryTabs         (v-model="currentCategory")
│   ├── ProgressBar          (:progress="progressPct")
│   ├── SceneCard            (:emoji, :description)
│   ├── ChineseSentence      (:text="question.chinese")
│   ├── EnglishInput         (v-model="userAnswer", ref for autofocus)
│   │   └── AnswerFeedback   (v-if="revealed", :correct, :expected)
│   ├── HintList             (collapsed by default, toggleable)
│   └── ActionBar
│       ├── RevealButton     (toggles `revealed`)
│       ├── AnalysisButton   (opens AnalysisView; shown when revealed)
│       ├── NextButton       (advances to next question)
│       ├── MarkKnownButton  (adds to `known`)
│       └── MarkReviewButton (adds to `review`)
└── AnalysisView             (v-else, :question)
    ├── Header               (back button + "✓ 答对了" / "已查看解析")
    ├── EnglishSentence      (large, primary color)
    ├── WordBreakdownTable   (5 rows × N words: text/phonetic/POS/meaning/role)
    ├── UsageContext         (💡 适用场景 — when/where to use)
    ├── ExamplesList         (💬 示例 — 1–2 dialogue examples)
    ├── SimilarSentences     (📝 相似句式 — 2 same-pattern alternatives)
    └── NextButton           (returns to practice view with next question)
```

State managed in `App` via `ref()` / `reactive()`:

```js
const currentCategory = ref('daily_greetings')
const questions = ref(QUESTIONS)        // from inline JSON
const index = ref(0)                     // index within filtered list
const userAnswer = ref('')
const revealed = ref(false)
const showAnalysis = ref(false)         // toggles PracticeView ↔ AnalysisView
const progress = ref(loadProgress())
```

---

## 9. User Flows

### Flow 1: Default practice loop (correct answer)

1. Page loads → restore `lastCategory` from localStorage
2. Show first unmastered question in that category
3. User types English answer
4. User clicks "下一题" (or presses Enter)
5. **If answer is correct** → switch to `AnalysisView` automatically
6. User reviews word breakdown + similar sentences
7. User clicks "下一题" → back to `PracticeView` with next question

### Flow 2: Wrong / empty answer

1. User types English answer (or leaves empty)
2. User clicks "下一题" (or presses Enter)
3. If answer is empty → just advance to next question, no analysis
4. If answer is non-empty but wrong → show "✗ 答案不正确" inline, advance to next question (still no auto-analysis — user must opt in)
5. After advancing, user can no longer see this question's analysis for the session (it'll come back in review)

### Flow 3: Reveal answer

1. User clicks "显示答案" (or presses Tab)
2. Reveal mode toggles on
3. Expected English shown below input in green
4. User's typed text compared (case-insensitive, punctuation-insensitive)
5. ✓ green if match, ✗ red with diff highlight if not
6. "查看解析" button appears

### Flow 4: Mark progress

1. User clicks "已掌握" → question id added to `known`, auto-advance to next question
2. User clicks "需复习" → question id added to `review`, auto-advance to next question
3. Progress bar updates immediately
4. Persisted to localStorage on every change

### Flow 5: Category switch

1. Click category tab
2. Reset `index` to 0
3. Filter to that category
4. Save `lastCategory`

---

## 10. Answer Comparison Logic

```js
function normalize(s) {
  return s
    .toLowerCase()
    .trim()
    .replace(/[.,!?;:'"]/g, '')   // strip common punctuation
    .replace(/\s+/g, ' ')         // collapse whitespace
}

function isCorrect(input, expected) {
  return normalize(input) === normalize(expected)
}
```

Acceptance criteria:
- `"How have you been?"` matches `"how have you been"`
- `"how have you been  "` matches `"how have you been"`
- Case-insensitive
- Trailing/leading whitespace ignored
- Common punctuation ignored

Strict match (not fuzzy) — user can always click "显示答案" to see expected form.

---

## 11. Question Selection Algorithm

When category changes or user marks known/review:

```js
function selectNext(list, known, review) {
  // Priority: unseen/review first, then known last
  const unseen = list.filter(q => !known.includes(q.id) && !review.includes(q.id))
  const toReview = list.filter(q => review.includes(q.id))
  const pool = [...unseen, ...toReview]
  return pool[index.value % pool.length] || list[0]
}
```

- If all marked known → loop back to start of list (encourage review)
- If list empty → fallback to first question in category

---

## 12. Keyboard Shortcuts

| Key | Action |
|---|---|
| `Enter` | Submit answer / advance to next question |
| `Tab` | Toggle reveal answer |
| `1`–`6` | Switch to category 1–6 |
| `/` | Focus the English input |

Shortcuts disabled when user is typing in the input (except `Tab` which moves focus naturally — we override `Enter` only).

---

## 13. Styling

- Dark mode by default (matches the livestream aesthetic from screenshot)
- Accent color: emerald green `#10b981` (pairs well with dark backgrounds)
- Background: deep blue-black `#0f1419`
- Text: off-white `#e8e8e8`
- Card surfaces: `#1a2028`
- Success: `#22c55e` (brighter green for correct answer feedback)
- Error: `#f87171`
- Font stack: system-ui + CJK fallback (`-apple-system, "PingFang SC", "Microsoft YaHei"`)

No external fonts, no icon libraries — emoji handles all icons.

---

## 14. Error Handling

| Failure | Behavior |
|---|---|
| localStorage disabled (private mode) | Wrap writes in try/catch; app still works in-memory |
| Question list empty | Show "暂无题目" placeholder |
| Invalid category in localStorage | Fallback to `daily_greetings` |
| User answers with only whitespace | Treated as empty; "下一题" just advances |

No network failures possible (no network calls).

---

## 15. Accessibility

- All interactive elements keyboard-reachable
- Input has visible focus ring
- Color is not the only signal — ✓/✗ icons also shown
- ARIA labels on icon-only buttons
- Semantic HTML (`<button>`, `<nav>`, `<main>`)
- Min font size 16px on mobile

---

## 16. Testing Strategy

Given the single-file, no-build nature, testing is **manual + scripted**:

1. **Manual smoke test:** Double-click `index.html` in Safari/Chrome/Firefox; verify all 6 categories load, answer check works, progress persists across reload.
2. **Manual keyboard test:** Enter / Tab / number keys work.
3. **Manual mobile test:** Open via `file://` on phone (AirDrop the file), verify layout.
4. **GitHub Pages test:** Push to repo, enable Pages, verify at `https://<user>.github.io/english-practice/`.
5. **Data sanity:** Open DevTools, inspect inline JSON, verify all 120 questions have non-empty `english` and `chinese`.

No automated test framework in v1 — explicitly out of scope.

---

## 17. Deployment

### Initial setup (one-time)
1. Create GitHub repo `english-practice`
2. Push `index.html` and `README.md` to `main`
3. Repo Settings → Pages → Source: "Deploy from a branch" → `main` / `(root)`
4. Wait ~1 min; site live at `https://<user>.github.io/english-practice/`

### Updating content
- Edit inline JSON in `index.html` (or ask AI to regenerate and paste in)
- `git add index.html && git commit -m "..." && git push`
- Pages redeploys automatically

### Estimated inline JSON size
With 120 questions × (~7 words × 7 fields + usage_context + 2 examples × 2 fields + 2 similar × 2 fields), the inline JSON is approximately **100–140 KB** (uncompressed). This is well within practical limits for a single HTML file and won't noticeably affect page load.

---

## 18. Future Work (Post-v1)

### Listening Practice (听写练习) — v2 candidate
- Use Web Speech API (`speechSynthesis`) to read the English sentence aloud
- Three modes:
  - **听写**：play audio N times, user types what they hear, compare to `question.english`
  - **听选**：play audio, show 3 candidate Chinese sentences, user picks the matching one
  - **跟读**：play audio, user repeats into mic, show waveform + replay their recording
- No backend needed — entirely client-side
- Browser compat: Chrome works fully; Safari has partial TTS support; mobile Safari limited

### Speaking Practice (说写练习) — v2 candidate
- Use Web Speech API (`SpeechRecognition`) to capture user's voice
- User reads the English sentence aloud, system transcribes and scores
- Compare transcribed text to `question.english` (case + punctuation insensitive)
- Show pronunciation feedback (basic — no phoneme-level scoring)
- Caveat: `SpeechRecognition` is Chrome-only and requires HTTPS (works on GitHub Pages, NOT on `file://`)

### Spaced Repetition (SRS)
- Track `lastSeen`, `lastCorrect`, `ease` per question in localStorage
- Surface "review" questions more often based on simple Leitner system or SM-2

### Other
- Import/export progress JSON (for backup / sync between devices via copy-paste)
- Generate fresh question batches via AI (paste output into `index.html`)
- English UI strings (currently mixed CN/EN by design)
- Search by English word across all questions
