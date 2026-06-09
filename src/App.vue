<script setup>
import { ref, computed } from 'vue'

const SYSTEM_PROMPT = `Ты — эксперт по созданию вирусных Instagram-каруселей про искусственный интеллект, автоматизацию, нейросети, продуктивность и будущее технологий.

Твоя задача — создавать карусели, которые люди сохраняют, пересылают друзьям и дочитывают до последнего слайда.

Структура карусели:
- Слайд 1 — Хук (останавливает скролл: цифры, провокация, страх упустить, любопытство)
- Слайды 2–7 — Основные (одна мысль, 1 заголовок + 2–4 короткие строки)
- Последний слайд — Сильный вывод (CTA, вопрос, список)

Стиль: премиальный, современный, умный, минималистичный, без воды.

ЗАПРЕЩЕНО: "В современном мире", "Сегодня технологии развиваются", "Как известно".

Ответь ТОЛЬКО в формате JSON (без markdown, без backticks):
{
  "slides": [{ "title": "заголовок", "body": "текст через \\n" }],
  "post_title": "заголовок поста",
  "description": "описание для Instagram",
  "hashtags": ["хештег1","хештег2","хештег3","хештег4","хештег5","хештег6","хештег7","хештег8","хештег9","хештег10"],
  "visual_tips": ["совет 1", "совет 2"],
  "virality_score": 8,
  "virality_reason": "причина"
}

Создавай ровно 8 слайдов.`

const THEMES = [
  { label: 'AI-агенты', icon: '🤖' },
  { label: 'ChatGPT хаки', icon: '💡' },
  { label: 'Автоматизация бизнеса', icon: '⚙️' },
  { label: 'Профессии под угрозой', icon: '⚠️' },
  { label: 'Заработок с ИИ', icon: '💰' },
  { label: 'Midjourney', icon: '🎨' },
  { label: 'ElevenLabs', icon: '🎙️' },
  { label: 'Продуктивность', icon: '🚀' },
]

const SLIDE_ACCENTS = [
  '#FF3B30','#FF9500','#FFCC00','#34C759','#00C7BE','#007AFF','#AF52DE','#FF2D55',
]

const apiKey = ref(localStorage.getItem('carousel_api_key') || '')
const showKeyInput = ref(!localStorage.getItem('carousel_api_key'))
const topic = ref('')
const loading = ref(false)
const data = ref(null)
const error = ref('')
const activeSlide = ref(0)
const tab = ref('preview')

function saveApiKey() {
  if (apiKey.value.trim()) {
    localStorage.setItem('carousel_api_key', apiKey.value.trim())
    showKeyInput.value = false
    error.value = ''
  }
}
function resetApiKey() {
  localStorage.removeItem('carousel_api_key')
  apiKey.value = ''
  showKeyInput.value = true
  data.value = null
}

const slides = computed(() => data.value?.slides || [])
const accent = computed(() => SLIDE_ACCENTS[activeSlide.value % SLIDE_ACCENTS.length])

async function generate(inputTopic) {
  const t = inputTopic || topic.value
  if (!t.trim()) return
  loading.value = true
  error.value = ''
  data.value = null
  activeSlide.value = 0
  try {
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': apiKey.value.trim(),
        'anthropic-version': '2023-06-01',
        'anthropic-dangerous-direct-browser-access': 'true',
      },
      body: JSON.stringify({
        model: 'claude-sonnet-4-6',
        max_tokens: 4000,
        system: SYSTEM_PROMPT,
        messages: [{ role: 'user', content: `Создай карусель на тему: ${t}` }],
      }),
    })
    const json = await res.json()
    if (!res.ok) throw new Error(`API ${res.status}: ${json?.error?.message || JSON.stringify(json)}`)
    const raw = json.content?.map(b => b.text || '').join('') || ''
    const clean = raw.replace(/```json|```/g, '').trim()
    data.value = JSON.parse(clean)
    tab.value = 'preview'
  } catch (e) {
    error.value = 'Ошибка: ' + (e?.message || String(e))
  } finally {
    loading.value = false
  }
}

function handleTheme(label) {
  topic.value = label
  generate(label)
}

function copyText(text) {
  navigator.clipboard.writeText(text)
}
</script>

<template>
  <div class="app">
    <header class="header">
      <span class="header-logo">CAROUSEL.AI</span>
      <div class="header-right">
        <span v-if="!showKeyInput" class="key-badge" title="API ключ сохранён">
          <span class="key-dot"></span> API ключ активен
        </span>
        <button v-if="!showKeyInput" class="key-reset-btn" @click="resetApiKey">Сменить ключ</button>
        <span v-else class="header-sub">Instagram Generator</span>
      </div>
    </header>

    <!-- API KEY SCREEN -->
    <div v-if="showKeyInput" class="key-screen">
      <div class="key-card">
        <div class="key-icon">🔑</div>
        <h2 class="key-title">Введи Anthropic API ключ</h2>
        <p class="key-desc">
          Ключ сохраняется локально в браузере и никуда не передаётся.<br>
          Получить ключ: <a href="https://console.anthropic.com" target="_blank" class="key-link">console.anthropic.com</a>
        </p>
        <div class="key-input-row">
          <input
            class="input-field"
            v-model="apiKey"
            type="password"
            placeholder="sk-ant-..."
            @keydown.enter="saveApiKey"
          />
          <button class="gen-btn" @click="saveApiKey" :disabled="!apiKey.trim()">
            → Сохранить
          </button>
        </div>
      </div>
    </div>

    <main class="main" v-if="!showKeyInput">
      <div class="input-section">
        <h1 class="main-title">
          Генератор вирусных<br>
          <span class="title-muted">каруселей для Instagram</span>
        </h1>
        <p class="main-desc">Введи тему → получи готовую карусель с превью и метаданными</p>

        <div class="input-row">
          <input
            class="input-field"
            v-model="topic"
            placeholder="Например: AI-агенты, автоматизация продаж, Runway ML..."
            @keydown.enter="generate()"
          />
          <button class="gen-btn" @click="generate()" :disabled="loading || !topic.trim()">
            {{ loading ? '...' : '→ Создать' }}
          </button>
        </div>

        <div class="themes-row">
          <button
            v-for="t in THEMES" :key="t.label"
            class="theme-chip"
            @click="handleTheme(t.label)"
            :disabled="loading"
          >
            <span>{{ t.icon }}</span> {{ t.label }}
          </button>
        </div>
      </div>

      <div v-if="error" class="error-box">{{ error }}</div>

      <div v-if="loading" class="loading-state">
        <span class="loading-text">ГЕНЕРИРУЮ КАРУСЕЛЬ...</span>
      </div>

      <div v-if="data && !loading" class="results">
        <div class="tabs">
          <button :class="['tab-btn', tab === 'preview' ? 'active' : 'inactive']" @click="tab = 'preview'">Превью слайдов</button>
          <button :class="['tab-btn', tab === 'meta' ? 'active' : 'inactive']" @click="tab = 'meta'">Мета и публикация</button>
        </div>

        <!-- PREVIEW -->
        <div v-if="tab === 'preview'" class="preview-layout">
          <div class="slide-area">
            <div class="slide-card" :key="activeSlide" :style="{ border: `1px solid ${accent}22`, boxShadow: `0 0 60px ${accent}18, 0 0 0 1px #1a1a1a` }">
              <div class="slide-accent-line" :style="{ background: accent }"></div>
              <div class="slide-num" :style="{ color: accent + '88' }">
                {{ String(activeSlide + 1).padStart(2,'0') }} / {{ String(slides.length).padStart(2,'0') }}
              </div>
              <div class="slide-content">
                <div class="slide-title">{{ slides[activeSlide]?.title }}</div>
                <div class="slide-body">{{ slides[activeSlide]?.body }}</div>
              </div>
              <div class="slide-footer">
                <div class="slide-dot-accent" :style="{ background: accent }"></div>
                <span class="slide-brand">CAROUSEL.AI</span>
              </div>
            </div>

            <div class="slide-nav">
              <button class="nav-btn" @click="activeSlide--" :disabled="activeSlide === 0">←</button>
              <div class="dots-row">
                <div
                  v-for="(_, i) in slides" :key="i"
                  :class="['dot', i === activeSlide ? 'dot-active' : '']"
                  :style="i === activeSlide ? { background: SLIDE_ACCENTS[i % SLIDE_ACCENTS.length], width: '24px' } : {}"
                  @click="activeSlide = i"
                ></div>
              </div>
              <button class="nav-btn" @click="activeSlide++" :disabled="activeSlide === slides.length - 1">→</button>
            </div>
          </div>

          <div class="slide-list">
            <div class="slide-list-label">ВСЕ СЛАЙДЫ</div>
            <div
              v-for="(slide, i) in slides" :key="i"
              :class="['slide-list-item', i === activeSlide ? 'slide-list-item-active' : '']"
              @click="activeSlide = i"
            >
              <div class="slide-list-num" :style="i === activeSlide ? { background: SLIDE_ACCENTS[i % SLIDE_ACCENTS.length], color: '#000' } : {}">{{ i + 1 }}</div>
              <div class="slide-list-title" :style="{ color: i === activeSlide ? '#f0f0f0' : '#888' }">{{ slide.title }}</div>
            </div>
          </div>
        </div>

        <!-- META -->
        <div v-if="tab === 'meta'" class="meta-tab">
          <div class="meta-card">
            <div class="meta-card-header">
              <span class="meta-label">Рейтинг вирусности</span>
              <span class="virality-score">{{ data.virality_score }}/10</span>
            </div>
            <div class="score-track"><div class="score-fill" :style="{ width: `${data.virality_score * 10}%` }"></div></div>
            <p class="meta-text">{{ data.virality_reason }}</p>
          </div>

          <div class="meta-card">
            <div class="meta-card-header">
              <span class="meta-label">Заголовок поста</span>
              <button class="copy-btn" @click="copyText(data.post_title)">Копировать</button>
            </div>
            <p class="meta-value">{{ data.post_title }}</p>
          </div>

          <div class="meta-card">
            <div class="meta-card-header">
              <span class="meta-label">Описание Instagram</span>
              <button class="copy-btn" @click="copyText(data.description)">Копировать</button>
            </div>
            <p class="meta-text">{{ data.description }}</p>
          </div>

          <div class="meta-card">
            <div class="meta-card-header">
              <span class="meta-label">Хештеги</span>
              <button class="copy-btn" @click="copyText(data.hashtags.join(' '))">Копировать все</button>
            </div>
            <div class="hashtags-row">
              <span v-for="(h, i) in data.hashtags" :key="i" class="hashtag-pill">{{ h }}</span>
            </div>
          </div>

          <div class="meta-card">
            <span class="meta-label" style="display:block;margin-bottom:12px">Идеи визуального оформления</span>
            <div class="tips-list">
              <div v-for="(tip, i) in data.visual_tips" :key="i" class="tip-row">
                <div class="tip-num" :style="{ background: SLIDE_ACCENTS[i%SLIDE_ACCENTS.length]+'22', border: `1px solid ${SLIDE_ACCENTS[i%SLIDE_ACCENTS.length]}44`, color: SLIDE_ACCENTS[i%SLIDE_ACCENTS.length] }">{{ i + 1 }}</div>
                <span class="tip-text">{{ tip }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-if="!data && !loading" class="empty-state">
        <div class="empty-icon">⚡</div>
        <p class="empty-text">Введи тему или выбери быстрый пресет выше</p>
      </div>
    </main>

    <footer class="footer">
      <span>Powered by Claude 4.1</span>
      <span>Instagram Carousel Generator</span>
    </footer>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;700&family=Syne:wght@700;800&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
:root{--bg:#0a0a0a;--bg2:#111;--bg3:#141414;--ink:#f0f0f0;--ink2:#888;--ink3:#444;--border:#222}
body{background:var(--bg);color:var(--ink);font-family:'DM Sans','Helvetica Neue',sans-serif;font-weight:300;min-height:100vh}
::-webkit-scrollbar{width:4px}::-webkit-scrollbar-track{background:#111}::-webkit-scrollbar-thumb{background:#333;border-radius:2px}
.app{display:flex;flex-direction:column;min-height:100vh}
.header{border-bottom:1px solid var(--border);padding:16px 24px;display:flex;justify-content:space-between;align-items:center}
.header-logo{font-family:'Syne',sans-serif;font-weight:800;font-size:18px;letter-spacing:-0.5px;background:linear-gradient(90deg,#fff 60%,#666);-webkit-background-clip:text;-webkit-text-fill-color:transparent}
.header-sub{font-size:12px;color:var(--ink3)}
.main{flex:1;padding:32px 24px;max-width:900px;margin:0 auto;width:100%}
.input-section{margin-bottom:40px}
.main-title{font-family:'Syne',sans-serif;font-size:clamp(24px,5vw,42px);font-weight:800;line-height:1.1;letter-spacing:-1px;margin-bottom:8px}
.title-muted{color:var(--ink3)}
.main-desc{color:var(--ink3);font-size:14px;margin-bottom:24px}
.input-row{display:flex;gap:10px;margin-bottom:16px}
.input-field{flex:1;background:var(--bg3);border:1px solid var(--border);color:var(--ink);font-family:'DM Sans',sans-serif;font-size:15px;padding:14px 18px;border-radius:12px;outline:none;transition:border-color 0.2s}
.input-field:focus{border-color:#444}
.input-field::placeholder{color:var(--ink3)}
.gen-btn{background:var(--ink);color:var(--bg);border:none;font-family:'Syne',sans-serif;font-weight:700;font-size:14px;letter-spacing:0.5px;padding:14px 28px;border-radius:12px;cursor:pointer;white-space:nowrap;transition:opacity 0.15s,transform 0.1s}
.gen-btn:hover{opacity:.85;transform:translateY(-1px)}
.gen-btn:disabled{opacity:.3;cursor:not-allowed;transform:none}
.themes-row{display:flex;gap:8px;flex-wrap:wrap}
.theme-chip{background:var(--bg3);border:1px solid var(--border);color:#aaa;font-size:13px;font-family:'DM Sans',sans-serif;padding:8px 14px;border-radius:100px;cursor:pointer;display:flex;align-items:center;gap:6px;transition:all 0.15s}
.theme-chip:hover{border-color:#444;color:var(--ink)}
.error-box{background:#1a0a0a;border:1px solid #3a1a1a;border-radius:12px;padding:14px 18px;color:#ff6b6b;font-size:14px;margin-bottom:24px}
.loading-state{text-align:center;padding:60px 0}
.loading-text{font-family:'Syne',sans-serif;font-size:14px;color:var(--ink3);letter-spacing:2px;animation:pulse 1.2s ease-in-out infinite}
@keyframes pulse{0%,100%{opacity:1}50%{opacity:.4}}
.results{}
.tabs{display:flex;gap:4px;margin-bottom:28px;background:#0f0f0f;padding:4px;border-radius:100px;width:fit-content}
.tab-btn{font-family:'DM Sans',sans-serif;font-size:13px;font-weight:500;padding:8px 20px;border-radius:100px;border:none;cursor:pointer;transition:all 0.15s}
.tab-btn.active{background:var(--ink);color:var(--bg)}
.tab-btn.inactive{background:transparent;color:#666}
.tab-btn.inactive:hover{color:#aaa}
.preview-layout{display:flex;gap:32px;align-items:flex-start;flex-wrap:wrap}
.slide-area{display:flex;flex-direction:column;align-items:center;gap:20px}
.slide-card{width:100%;max-width:380px;aspect-ratio:4/5;border-radius:20px;background:#0e0e0e;display:flex;flex-direction:column;justify-content:space-between;padding:36px 32px;position:relative;overflow:hidden;animation:slideIn 0.3s ease}
@keyframes slideIn{from{opacity:0;transform:translateX(20px)}to{opacity:1;transform:translateX(0)}}
.slide-accent-line{position:absolute;top:0;left:32px;right:32px;height:2px;border-radius:0 0 2px 2px}
.slide-num{position:absolute;top:20px;right:24px;font-family:'Syne',sans-serif;font-size:12px;font-weight:700}
.slide-content{flex:1;display:flex;flex-direction:column;justify-content:center;padding-top:20px}
.slide-title{font-family:'Syne',sans-serif;font-weight:800;font-size:clamp(18px,3.5vw,22px);line-height:1.2;letter-spacing:-0.5px;color:var(--ink);margin-bottom:16px}
.slide-body{font-size:14px;line-height:1.7;color:#888;font-weight:300;white-space:pre-line}
.slide-footer{display:flex;align-items:center;gap:8px;padding-top:16px;border-top:1px solid #1a1a1a}
.slide-dot-accent{width:6px;height:6px;border-radius:50%}
.slide-brand{font-size:11px;color:#444;letter-spacing:1px;font-weight:500}
.slide-nav{display:flex;align-items:center;gap:12px}
.dots-row{display:flex;gap:6px;align-items:center}
.dot{width:8px;height:8px;border-radius:50%;background:#333;cursor:pointer;transition:all 0.2s;flex-shrink:0}
.dot-active{border-radius:4px}
.nav-btn{background:#1a1a1a;border:1px solid var(--border);color:var(--ink);width:36px;height:36px;border-radius:50%;display:flex;align-items:center;justify-content:center;cursor:pointer;font-size:16px;transition:all 0.15s}
.nav-btn:hover{background:#222}
.nav-btn:disabled{opacity:.2;cursor:not-allowed}
.slide-list{flex:1;min-width:240px}
.slide-list-label{font-size:11px;color:var(--ink3);letter-spacing:1px;margin-bottom:12px;font-weight:500}
.slide-list-item{background:transparent;border:1px solid transparent;border-radius:10px;padding:12px 14px;cursor:pointer;display:flex;gap:12px;align-items:flex-start;transition:all 0.15s;margin-bottom:6px}
.slide-list-item-active{background:var(--bg3);border-color:var(--border)}
.slide-list-num{width:22px;height:22px;border-radius:6px;background:#1a1a1a;display:flex;align-items:center;justify-content:center;font-size:10px;font-family:'Syne',sans-serif;font-weight:700;color:#555;flex-shrink:0;transition:all 0.15s}
.slide-list-title{font-size:13px;font-weight:500;line-height:1.3}
.meta-tab{display:flex;flex-direction:column;gap:14px}
.meta-card{background:var(--bg2);border:1px solid #1e1e1e;border-radius:14px;padding:20px}
.meta-card-header{display:flex;justify-content:space-between;align-items:center;margin-bottom:10px}
.meta-label{font-size:11px;font-weight:500;letter-spacing:1px;color:#555;text-transform:uppercase}
.meta-value{font-size:15px;font-weight:500;color:#ddd;line-height:1.4}
.meta-text{font-size:13px;color:#666;line-height:1.6;font-weight:300}
.copy-btn{background:#1e1e1e;border:none;color:#888;font-size:11px;font-family:'DM Sans',sans-serif;padding:5px 12px;border-radius:6px;cursor:pointer;transition:all 0.15s}
.copy-btn:hover{background:#252525;color:#ccc}
.virality-score{font-family:'Syne',sans-serif;font-weight:800;font-size:20px;color:#FF9500}
.score-track{background:#1a1a1a;border-radius:100px;height:4px;margin-bottom:10px}
.score-fill{height:100%;border-radius:100px;background:linear-gradient(90deg,#FF3B30,#FF9500,#FFCC00);transition:width 0.8s cubic-bezier(0.16,1,0.3,1)}
.hashtags-row{display:flex;flex-wrap:wrap;gap:6px}
.hashtag-pill{background:var(--bg3);border:1px solid #1e1e1e;color:#888;font-size:12px;padding:4px 10px;border-radius:100px}
.tips-list{display:flex;flex-direction:column;gap:8px}
.tip-row{display:flex;gap:10px;align-items:flex-start}
.tip-num{width:20px;height:20px;border-radius:4px;display:flex;align-items:center;justify-content:center;font-size:9px;font-weight:700;flex-shrink:0;font-family:'Syne',sans-serif}
.tip-text{font-size:13px;color:#777;line-height:1.5;font-weight:300}
.empty-state{border:1px dashed #1a1a1a;border-radius:20px;padding:60px 32px;text-align:center}
.empty-icon{font-size:48px;margin-bottom:16px;opacity:.3}
.empty-text{color:#333;font-size:14px}
.footer{border-top:1px solid var(--border);padding:14px 24px;display:flex;justify-content:space-between;align-items:center;font-size:11px;color:var(--ink3)}
@media(max-width:600px){.preview-layout{flex-direction:column}.slide-card{max-width:100%}}
.header-right{display:flex;align-items:center;gap:12px}
.key-badge{font-size:12px;color:#30D158;display:flex;align-items:center;gap:6px;font-family:'DM Sans',sans-serif}
.key-dot{width:6px;height:6px;border-radius:50%;background:#30D158;display:inline-block}
.key-reset-btn{background:transparent;border:1px solid var(--border);color:var(--ink3);font-size:12px;font-family:'DM Sans',sans-serif;padding:5px 12px;border-radius:8px;cursor:pointer;transition:all 0.15s}
.key-reset-btn:hover{border-color:#444;color:var(--ink)}
.key-screen{flex:1;display:flex;align-items:center;justify-content:center;padding:40px 24px}
.key-card{background:var(--bg2);border:1px solid var(--border);border-radius:20px;padding:40px;max-width:480px;width:100%;text-align:center}
.key-icon{font-size:40px;margin-bottom:20px}
.key-title{font-family:'Syne',sans-serif;font-size:24px;font-weight:800;letter-spacing:-0.5px;margin-bottom:12px}
.key-desc{font-size:14px;color:var(--ink3);line-height:1.6;margin-bottom:28px}
.key-link{color:#007AFF;text-decoration:none}
.key-link:hover{text-decoration:underline}
.key-input-row{display:flex;gap:10px}
</style>
