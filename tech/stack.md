# PÄSH · Tech Stack
> Tier: evolving. Технологический стек и интеграции.
> Разработчики: Amir (сайт, интеграции), Станислав (архитектура, боты).

---

## Архитектура

```
pash.kz (сайт)
    ↓ TipTop Pay webhook
Railway (Python боты)
    ├── Мани-Пенни (WA + TG клиент)
    ├── М / Robin (TG internal + scheduled duties)
    └── Q (Vision + аналитика)
    ↓↑ read-only
Supabase (БД + API)
    ↑ read
pash-dashboard_2.html (Vercel)

GitHub
    └── pash-team-os (knowledge repo, read-only для агентов)
```

---

## База данных: Supabase

**URL:** `nzskcvwghqadvobjrppf.supabase.co`
**RLS:** включён

| Таблица | Содержание | Tier |
|---------|-----------|------|
| `products` | SKU, цены PÄSH, категории (hook/margin) | evolving |
| `price_snapshots` | Цены конкурентов (магазин, лавка, базар, алтын-орда) | volatile |
| `alerts` | Ценовые алерты (seen/unseen) | volatile |
| `orders` | Заказы покупателей, статусы, ПВЗ | volatile |
| `users` | Покупатели (anonymized) | evolving |
| `price_queries` | История запросов цен | volatile |
| `micro_districts` | 50 микрорайонов Алматы с центроидами | stable |

**Ключи:**
- `SUPABASE_ANON_KEY` — для публичного чтения (дашборд, агенты)
- `SUPABASE_SERVICE_KEY` — только бэкенд, никогда в агентах!

---

## API

**Base URL:** `https://api.pash.kz/api`
**Документация:** запросить у Amir (актуальная версия не в репо)

---

## Платежи: TipTop Pay

**Провайдер:** TipTop Pay (БЦК)
**Комиссия:** 3% эквайринг
**Webhook → Railway → Supabase** (обновление статуса заказа)
**Secret:** `TIPTOP_WEBHOOK_SECRET` (Railway env)

---

## AI / ML

| Инструмент | Использование | Ключ |
|-----------|--------------|------|
| Claude API (claude-sonnet-4-6) | М (Robin), все агенты | `ANTHROPIC_API_KEY` |
| Gemini API (Vision) | Q (контроль стеллажей, парсинг чеков) | у Erdos |

---

## Мессенджеры

| Канал | Использование | Bot/SIM |
|-------|-------------|---------|
| WhatsApp SIM#1 | Мани-Пенни (клиенты) | WA Business API (ждём апрув Meta) |
| WhatsApp SIM#2 | М, Q, команда (внутренний) | WA Business API |
| Telegram @pash_client_price_bot | Мани-Пенни + краудсорсинг цен | Bot API |
| Telegram internal | М, команда | Bot API (`TG_BOT_TOKEN_M`) |
| @pash_club | Первые покупатели | read-only для М (VoC) |
| @pash_channel | Публичный канал | ручное ведение |

---

## Хостинг и деплой

| Сервис | Что | Примечания |
|--------|-----|-----------|
| Railway | Python-боты (М, Мани-Пенни, Q) | + Cron для Robin duties |
| Vercel | HTML-дашборды | `pash-dashboard_2.html` |
| GitHub | Knowledge repo (pash-team-os) | read-only для агентов |

---

## Дашборды

**Внутренний ("Bloomberg terminal"):**
- Файл: `pash-dashboard_2.html`
- Стек: vanilla JS + Supabase REST + IBM Plex Mono
- Данные: products, price_snapshots, alerts в реальном времени
- Dark/light theme toggle
- Деплой: Vercel

**Публичная карта цен:**
- Стек: Leaflet.js + CartoDB dark tiles
- ~110 точек данных
- Переключатель: РУС/КАЗ/ENG + день/ночь
- Деплой: Vercel

---

## Дизайн

**Canva:** проект логотипа ID `DAHCHhS_Lzo` (3 страницы)
**Важно:** Canva CDN не отдаёт изображения напрямую → скриншот для ревью в Claude

---

## Deferred Features (не сейчас)

- Парсинг чеков через Gemini Vision (нужны `receipt_id`, `is_promo`, таблицы `store_chains`, `store_locations`)
- QR-подтверждение выдачи водителем
- Автоматические уведомления покупателям (ботовая итерация 2, уточнить у Amir)
