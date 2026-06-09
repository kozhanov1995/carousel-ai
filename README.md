# Carousel.AI

**AI-генератор вирусных Instagram-каруселей на базе Claude API**

Вводишь тему → получаешь 8 готовых слайдов, описание, хештеги и рейтинг вирусности за ~10 секунд.

---

## Что умеет

- Генерирует **8 слайдов** по заданной теме с хуком, основными слайдами и CTA
- Показывает **визуальный превью** каждого слайда прямо в браузере
- Выдаёт **заголовок поста**, описание для Instagram и **10 хештегов** (с кнопкой копировать)
- Даёт **рейтинг вирусности** от 1 до 10 с объяснением
- Предлагает **идеи визуального оформления** для каждого слайда
- 8 быстрых **пресетов тем** для одного клика
- Сохраняет API ключ в `localStorage` — вводишь один раз

---

## Стек

| Слой | Технология |
|------|-----------|
| Frontend | Vue 3 + Composition API (`script setup`) |
| Bundler | Vite |
| AI | Claude API (claude-sonnet-4) |
| Deploy | Vercel |

---

## Быстрый старт

```bash
# 1. Клонируй репозиторий
git clone https://github.com/username/carousel-ai.git
cd carousel-ai

# 2. Установи зависимости
npm install

# 3. Запусти локально
npm run dev
```

Открой `http://localhost:5173`, введи свой [Anthropic API ключ](https://console.anthropic.com) и начинай генерировать.

---

## Деплой на Vercel

```bash
npm install -g vercel
vercel --prod
```

Или перетащи папку проекта на [vercel.com](https://vercel.com) — деплой за 30 секунд без конфигов.

---

## Структура проекта

```
carousel-ai/
├── src/
│   ├── App.vue        # Весь UI и логика (Vue 3 SFC)
│   └── main.js        # Точка входа
├── index.html
├── vite.config.js
└── package.json
```

---

## Как это работает

```
Пользователь вводит тему
        ↓
Vue отправляет запрос в Claude API
        ↓
System prompt задаёт роль эксперта по вирусному контенту,
структуру из 8 слайдов и формат JSON-ответа
        ↓
Claude возвращает структурированный JSON
        ↓
Vue рендерит интерактивный превью слайдов
```

**Ключевые технические решения:**
- `anthropic-dangerous-direct-browser-access: true` — прямые запросы из браузера без прокси
- `max_tokens: 4000` — гарантирует что JSON не обрывается на середине
- Structured output через system prompt — модель отвечает только валидным JSON

---

## Локальная разработка

```bash
npm run dev      # dev-сервер на localhost:5173
npm run build    # production сборка в /dist
npm run preview  # превью production сборки
```

---

## Лицензия

MIT
