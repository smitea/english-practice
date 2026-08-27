# English Practice

A single-file Vue 3 web app for personal English practice. Translate Chinese sentences to English, then automatically see a sentence analysis: word-by-word breakdown (phonetic, part of speech, Chinese meaning, grammar role), usage context, realistic examples, and similar sentences.

## Features

- 120 starter questions across 6 categories: 寒暄 / 工作 / 旅行 / 购物 / 社交 / 生活
- Auto-shows analysis after correct answer (word breakdown + usage + examples + similar)
- LocalStorage progress tracking (已掌握 / 需复习)
- Dark mode UI with emerald green accent
- Keyboard shortcuts: `Enter` next, `Tab` reveal, `1-6` switch category, `Esc` exit analysis
- Works offline via `file://` protocol
- Free hosting on GitHub Pages

## Run locally

Just open `index.html` in any modern browser. No install, no build.

```bash
open index.html        # macOS
xdg-open index.html    # Linux
start index.html       # Windows
```

Or just double-click `index.html` in your file manager.

## Deploy to GitHub Pages

1. Create a new GitHub repo (e.g. `english-practice`).
2. Push this directory to `main`.
3. Repo **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: main / (root)**.
4. Wait ~1 minute. Site live at `https://<your-username>.github.io/english-practice/`.

## Keyboard shortcuts

| Key | Action |
|---|---|
| `Enter` | Submit answer / advance to next question (also works inside any slot) |
| `Space` | Inside a slot: commit word and advance. Outside slots: toggle reveal answer. |
| `1`–`6` | Switch to category 1–6 |
| `Esc` | Exit analysis view |

### Word-slot behavior

Each English word gets its own input. As you type, the slot's bottom border updates:

- ✅ Correct word → green border.
- ❌ Wrong word → red border (no shake, no popup — just the visual state).

To commit a word and move to the next slot, press **`Space`**:

- Empty slot → just advance.
- Correct word → advance to next slot.
- Wrong word → **underline shakes**, stay on current slot. Correct the word (delete + retype) then press Space again.
- On the last slot → Space submits the whole question.

`Backspace` on an empty slot moves focus back to the previous slot.

## How to add more questions

Open `index.html` and find the `<script type="application/json" id="questions-data">` block. Each question follows this schema:

```json
{
  "id": 121,
  "category": "daily_greetings",
  "scene_emoji": "🙋",
  "scene_desc": "跟邻居打招呼",
  "chinese": "早上好!",
  "english": "Good morning!",
  "hints": ["早上 morning", "好 good"],
  "difficulty": "beginner",
  "words": [
    { "text": "Good",   "phonetic": "/ɡʊd/",    "pos": "adjective", "pos_zh": "形容词", "meaning_zh": "好的",     "role_zh": "定语" },
    { "text": "morning","phonetic": "/ˈmɔːnɪŋ/","pos": "noun",      "pos_zh": "名词",   "meaning_zh": "早晨",     "role_zh": "主语" }
  ],
  "usage_context": "正式或友好的早晨问候,适合邻里、同事、陌生人之间。",
  "similar": [
    { "english": "Good afternoon!", "chinese": "下午好!" },
    { "english": "Good evening!",   "chinese": "晚上好!" }
  ],
  "examples": [
    { "english": "Good morning! How's your day going?", "chinese": "早上好!你今天过得怎么样?" }
  ]
}
```

Append your question to the array (use the next available `id`). The valid categories are:

| ID | Chinese Label |
|---|---|
| `daily_greetings` | 寒暄 |
| `work` | 工作 |
| `travel` | 旅行 |
| `shopping` | 购物 |
| `social_emotion` | 社交 |
| `daily_life` | 生活 |

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
- Web Speech API (planned for v2)

## Roadmap (post-v1)

- 🎧 **Listening practice** (听写): use Web Speech API to play audio, user types what they hear
- 🎤 **Speaking practice** (说写): use Speech Recognition API, user reads aloud, system scores
- 🔁 Spaced repetition (track `lastSeen`, `ease` per question)
- � Import/export progress JSON

## License

MIT
