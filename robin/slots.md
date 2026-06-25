# ROBIN · Implementation Slots (PASH)
> Конфигурация по ROBIN-SPEC v0.6. Каждый слот — задокументированное решение.
> Tier: evolving. Разработчик: Amir + Станислав.

---

## Slot 1 · Chat Platform
**Telegram.** Два контекста:
- Внутренний TG-чат команды (D1, D2, D3, командные вопросы)
- DM Erdos (D4 VoC, D5 стратобзор, staged learnings на ревью)

## Slot 2 · Agent Runtime
**Claude API** (`claude-sonnet-4-6`). Max tokens: 1 000 на дайджест, 2 000 на VoC/стратобзор.
Оркестратор: Python на Railway. Telegram Bot API для доставки.

## Slot 3 · Hosting + Scheduler
**Railway.** Scheduler: Railway Cron Jobs. Cron-выражения из `robin/duties.md` — они источник правды, не hardcode.

## Slot 4 · Storage
- **Знания:** GitHub репо `pash-team-os` (read-only для агента)
- **Learnings staged:** `robin/staged/` в том же репо (агент пишет через GitHub API)
- **Learnings promoted:** `robin/promoted/` (только после PR-апрува от Erdos)
- **Interaction log:** Supabase таблица `robin_interactions`

## Slot 5 · Knowledge Repo
**GitHub.** Репо: `pash-team-os`. Entry file: `CLAUDE.md`.
Агент клонирует как read-only mount при инициализации сессии. Sync: при каждом scheduled duty.

## Slot 6 · Read-only Data Sources
| Источник | Таблицы/контент | Binding |
|----------|----------------|---------|
| Supabase | products, price_snapshots, alerts, orders, users, micro_districts | SUPABASE_URL + SUPABASE_ANON_KEY |
| Telegram | @pash_club messages (read) | TG_BOT_TOKEN + getTelegramMessages() |

**КРИТИЧНО:** Только read-only. Supabase: только anon key для чтения. Никакого service role key в агенте.

## Slot 7 · Chat Identity Registry
**Разрешённые destinations:**
- `internal_tg_group`: внутренний чат команды (ID: задать в env)
- `dm_erdos`: DM Erdos (TG User ID: задать в env)

**НЕ разрешено без явного указания:**
- @pash_channel (публичный)
- @pash_club (клиентский)
- DM другим членам команды

## Slot 8 · Passive-Capture Scope
Логируется: все сообщения внутреннего TG-чата команды.
@pash_club — только через D4 (VoC duty), не пассивно.
**Команда уведомлена** о логировании внутреннего чата.

## Slot 9 · Duty Roster
Смотри `robin/duties.md`. 5 задач: D1 (утро), D2 (пост-рейс), D3 (алерты), D4 (VoC), D5 (стратобзор).
Планировщик Railway читает duties.md напрямую — cron из него, не hardcode.

## Slot 10 · Digest Cadence + Destination
| Duty | Cron | Destination |
|------|------|------------|
| D1 Утро | `45 7 * * 1-6` | internal_tg_group |
| D2 Рейс | `30 22 * * 1-6` | internal_tg_group |
| D3 Алерты | `*/30 8-22 * * *` | internal_tg_group |
| D4 VoC | `0 9 * * 1` | dm_erdos |
| D5 Стратобзор | `0 10 * * 0` | dm_erdos |
Timezone все: `Asia/Almaty`.

## Slot 11 · Staleness Budget
**7 дней** (вместо дефолтных 14). PÄSH движется быстро — данные недельной давности уже устарели.
Volatile-документы (assortment.md, pvz.md) — дисконтировать через 3 дня.

## Slot 12 · DM Idle Window
**30 минут** (вместо 60). Операционная среда интенсивная — рейсы, заказы, срочные вопросы.

## Slot 13 · Ambient Context Size
**N = 10** последних сообщений канала при упоминании группе.

## Slot 14 · DM Reseed Turn Count
**5 последних ходов** при возобновлении DM сессии после idle.

## Slot 15 · Context Isolation
Агент М запускается в изолированном Railway-сервисе.
Никаких личных MCP-серверов, глобальных инструкций хоста.
Стоимость: замерить при первом деплое (ref: $0.039 vs $0.83 без изоляции).

## Slot 16 · Feedback + Promotion
**Affordance:** TG реакции 👍/👎 на сообщения М.
**Отрицательная реакция → staged learning:**
  `robin/staged/YYYY-MM-DD_[topic].md` с шаблоном:
  ```
  date: YYYY-MM-DD
  trigger: [что вызвало реакцию]
  problem: [что было не так]
  rule: [как делать правильно]
  route: memory | skill | repo-pr
  ```
**Promotion routes (Erdos выбирает при ревью):**
  - `memory` → скопировать в `robin/promoted/`
  - `skill` → обновить соответствующий skill.md
  - `repo-pr` → создать PR в pash-team-os (М НЕ пишет в репо напрямую)

## Slot 17 · Secrets Location
**Railway Environment Variables. Никогда в репо.**
```
SUPABASE_URL
SUPABASE_ANON_KEY       # read-only
TG_BOT_TOKEN_M          # агент М
TG_INTERNAL_CHAT_ID
TG_ERDOS_DM_ID
TG_PASH_CLUB_ID         # read-only для VoC
ANTHROPIC_API_KEY
GITHUB_TOKEN_ROBIN      # только write в robin/staged/
```

## Slot 18–20 · Meeting Layer
**Не реализовано.** Отложено до M4 (стабильная learning loop). Возможный стек: Recall.ai + AssemblyAI.

## Slot 21 · Input Modalities
**Текст** — основной. **Фото** — для Q (Vision-контроль стеллажей). Голосовые — отложено.

## Slot 22 · Team Skills Consumption
Скиллы агентов хранятся в `robin/skills/*.md` (canonical location).
В Railway: thin pointers — trigger metadata + инструкция читать SKILL.md из репо-mount.
Никогда не дублировать контент скилла в конфиге планировщика.

---

## Acceptance Tests (per ROBIN-SPEC §8)

| Milestone | Тест | Статус |
|-----------|------|--------|
| M0 | Свежая сессия → правильный ответ на «что такое безубыток PÄSH» с цитатой файла | ⬜ |
| M1 | Член команды спрашивает в TG → ответ с цитатой файла | ⬜ |
| M2 | D1 посылается по расписанию и сохраняется в interaction_log | ⬜ |
| M3 | Упоминание в группе → ответ с контекстом последних 10 сообщений | ⬜ |
| M4 | 👎 → staged file создан → Erdos промоутирует → следующий ответ изменился | ⬜ |
| M5 | Meetings — отложено | ⏸ |

**Первый деплой — цель M0+M1 за 1 день.**
