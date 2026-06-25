# pash-team-os

**Единый источник правды для PÄSH.** Phygital e-grocery, Алматы, 2026.

Это knowledge repo по архитектуре [Team OS Toolkit](https://github.com/BayramAnnakov/team-os-toolkit).
Потребители: агент М (Robin), Мани-Пенни, Q, сайт pash.kz, Erdos, Станислав, Amir, 007, 009.

## Начать здесь → [CLAUDE.md](./CLAUDE.md)

```
pash-team-os/
├── CLAUDE.md          ← навигационный корень, читать первым
├── soul.md            ← идентичность агента М
├── glossary.md        ← все термины PÄSH
├── decisions.md       ← датированный журнал решений
├── team/
│   ├── roster.md      ← команда, роли, контакты
│   └── agents.md      ← 007-архитектура агентов
├── operations/
│   ├── reys.md        ← полный процесс рейса
│   ├── pvz.md         ← точки выдачи
│   └── assortment.md  ← топ-8 SKU, цены, роли
├── business/
│   ├── model.md       ← бизнес-модель, 3 слоя
│   └── financials.md  ← ключевые числа, breakeven
├── brand/
│   └── guide.md       ← цвета, шрифты, тональность
├── tech/
│   └── stack.md       ← Supabase, Railway, API, боты
└── robin/
    ├── duties.md      ← расписание задач М (cron)
    ├── slots.md       ← ROBIN-SPEC конфигурация
    ├── staged/        ← staged learnings (пишет М)
    └── promoted/      ← promoted learnings (апрувит Erdos)
```

## Правила

- Агенты читают этот репо, но **не пишут** (кроме `robin/staged/`)
- Изменения — только через PR
- Volatile-файлы старше 7 дней — проверять актуальность
- Secrets — никогда в репо. Railway Environment Variables.

---

*PÄSH.KZ · Birge arzan — Вместе дешевле · Алматы · 2026*
