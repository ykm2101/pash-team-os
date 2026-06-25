# ROBIN · Duty Roster
> Источник правды для планировщика Railway Cron.
> Изменения только через PR. Cron-выражения в Asia/Almaty (UTC+5).
> Tier: evolving.

---

## D1 · Утренний бриф

```
trigger_cron: "45 7 * * 1-6"
trigger_tz: "Asia/Almaty"
owner: erdos
destination: internal_tg_group
```

**Человеческое описание:** Пн–Сб, 07:45 Алматы. Перед рейсом.

**Inputs:**
- `Supabase:orders` WHERE date=today AND status IN ('confirmed','pending')
- `Supabase:orders` WHERE status='pending_pickup' (не забраны с прошлого рейса)
- `Supabase:alerts` WHERE seen=false
- `Supabase:price_snapshots` WHERE recorded_at > NOW()-interval '24h'

**Output template:**
```
PÄSH · Утро [DD.MM]

РЕЙС СЕГОДНЯ:
• Подтверждено: N заказов (~X₸)
• Не забрано с прошлого: N ⚠️  ← только если >0

ЦЕНЫ:
• Активных алертов: N [если >0: список топ-3]

ФОКУС:
• [1–2 строки из открытых приоритетов]
```

**Fail behavior:** При ошибке Supabase — отправить `⚠️ М: не смог получить данные Supabase. Проверь вручную.`

---

## D2 · Пост-рейс дайджест

```
trigger_cron: "30 22 * * 1-6"
trigger_tz: "Asia/Almaty"
owner: erdos
destination: internal_tg_group
```

**Человеческое описание:** Пн–Сб, 22:30 Алматы. После закрытия окна выдачи (22:00).

**Inputs:**
- `Supabase:orders` WHERE date=today (все статусы)
- `Supabase:orders` WHERE status='pending_pickup' AND date=today (не забраны сегодня)

**Output template:**
```
PÄSH · Рейс [DD.MM] · Итоги

РЕЗУЛЬТАТ:
• Заказов: N | Выдано: N | Не забрано: N
• Выручка: X₸ | AOV: X₸
• vs Безубыток (28): [+N избыток / -N дефицит]

ТОП-3 ПОЗИЦИИ:
• [SKU]: N шт/кг

СЛЕДУЮЩИЙ РЕЙС [дата]:
• Предзаказов сейчас: N
```

**Примечание:** Не забранные заказы → сигнал для Мани-Пенни (follow-up клиенту).

---

## D3 · Прайс-алерты

```
trigger_cron: "*/30 8-22 * * *"
trigger_tz: "Asia/Almaty"
owner: q_agent
destination: internal_tg_group
condition: "only_if_new_alerts"
```

**Человеческое описание:** Каждые 30 мин с 08:00 до 22:00. Только если есть новые алерты.

**Inputs:**
- `Supabase:alerts` WHERE seen=false ORDER BY created_at DESC

**Output template (только если alerts > 0):**
```
⚠️ ЦЕНЫ · N новых алертов

[Продукт]: наша X₸ → магазин Y₸ (Δ Z%)
[рекомендация: держать / скорректировать / проверить]
```

**Post-action:** Пометить обработанные alerts.seen=true.

---

## D4 · Еженедельный VoC (голос клиента)

```
trigger_cron: "0 9 * * 1"
trigger_tz: "Asia/Almaty"
owner: erdos
destination: dm_erdos
sensitivity: private
```

**ВАЖНО:** Данные клиентов — только в DM Erdos, НИКОГДА в группу.

**Человеческое описание:** Каждый понедельник, 09:00 Алматы.

**Inputs:**
- TG @pash_club: сообщения за последние 7 дней
- `Supabase:orders` WHERE created_at > NOW()-interval '7d'
- `Supabase:users` (для определения аккаунтов команды → исключить)

**Предобработка (обязательно):**
1. Исключить сообщения от аккаунтов команды (Erdos, Станислав, Amir и тест-аккаунты)
2. Дедуплицировать: >3 одинаковых сообщения от одного пользователя = 1 сигнал
3. Сообщить объём выборки: «N сообщений от M уникальных пользователей»

**Output template:**
```
PÄSH · VoC · Неделя [N]
[N сообщений · M уникальных пользователей]

КЛИЕНТСКИЕ ТЕМЫ:
1. [тема]: «[парафраз, без имён]» — N упоминаний
2. ...

МЕТРИКИ НЕДЕЛИ:
• Retention: X% (цель >50%)
• Новых: N | Вернулись: N
• AOV: X₸ (тренд ↑/↓/→)
• Хиты: [топ-3 SKU]

СИГНАЛЫ ДЛЯ АССОРТИМЕНТА:
• Просят: [...]
• Не берут: [...]
• Жалобы: [...]
```

---

## D5 · Воскресный стратобзор

```
trigger_cron: "0 10 * * 0"
trigger_tz: "Asia/Almaty"
owner: erdos
destination: dm_erdos
```

**Человеческое описание:** Каждое воскресенье, 10:00 Алматы.

**Inputs:**
- `Supabase:orders` агрегаты за прошлую неделю
- `Supabase:price_snapshots` тренды vs конкуренты
- `decisions.md` открытые (pending) решения

**Output template:**
```
PÄSH · Стратобзор · Нед [N]

МЕТРИКИ:
• Рейсов: N | Заказов: N | Выручка: X₸
• Retention: X% [vs цель 50%: ±]
• AOV: X₸ → Y₸ [тренд]
• Прогресс к безубытку: ±N заказов/рейс

ЦЕНОВАЯ ПОЗИЦИЯ:
• Лидируем по: [SKU, % преимущество]
• Риски: [SKU где маржа сузилась]

ОТКРЫТЫЕ РЕШЕНИЯ:
• [из decisions.md, статус pending]

ФОКУС НА НЕДЕЛЮ:
1. [приоритет]
2. [приоритет]
```

---

## Правила для всех duties

1. **Failure = log + notify.** Если duty не выполнен — отправить ошибку. Молчание = сбой.
2. **Стale-data warning.** Данные Supabase старше 2 часов → пометить `⚠️ возможно устаревшие`.
3. **Destination follows sensitivity.** Данные клиентов — ТОЛЬКО в DM. Никогда в группу.
4. **Числа:** `140 000₸`, не `140000₸`. Всегда.
5. **Язык:** русский.
6. **Максимум:** D1/D2/D3 — 15 строк. D4/D5 — 30 строк. Больше = лишнее.
