# crypto-digest — Backlog & Architecture Plan (draft)

> Status: DRAFT — собрано в ходе архитектурного интервью. Не утверждено, не исполнено.
> Может измениться под более конкретные указания заказчика.
> Last updated: 2026-07-05

## 1. Продукт

Автоматический дайджест. Ниша: **Crypto & Blockchain: Business, Regulations, Engineering** (без под-фокусов).
Голос: **CTO software-компании** — разговорно-профессиональный, opinionated, факты вместо хайпа.
Язык: **US-English**. Стек: Astro (статик) → деплой на Vercel. Обложки: Replicate. Поиск: Tavily.

## 2. Design ledger (закрытые решения)

- Оркестрация: автономный серверный прогон без человека; headless Claude Code (переиспользуем сабагентов news-scout/writer/cover-artist/page-builder + MCP Tavily/Replicate).
- Триггер: периодический cron (не спящий процесс). Объём: 3 статьи/сутки для старта.
- Vercel = только деплой сайта (по push в master). Генерация живёт в раннере (TBD).
- Стиль: голос CTO; жёсткий лимит «25 слов на предложение» СНЯТ → «варьируй длину, без run-on».
- Заголовок ≤ 60 символов (совпадает со Stop-хуком validate-article.js). Description ≤ 160. Тело 300–500 слов.
- Единый источник правды для правил (лимиты + тема) — ВВОДИМ.
- Легаси: удалить 3 русские AI-статьи + sample-digest-post.md.
- Переименование проекта → crypto-digest (repo/package/consts).
- Тестовая инфраструктура: добавить Vitest, вынести чистую логику хуков в тестируемые модули (TDD-first).

## 3. Открытые вопросы (TBD, не блокируют общую архитектуру)

- РАННЕР: GitHub Actions cron vs VPS — ждём решения.
- ПОЛЕ ОБЛОЖКИ (#2 ниже): (a) внешний URL Replicate `cover: z.string().url()` + рендер cover в layout; vs (b) скачивание в src/assets + local heroImage. Ждём решения.

## 4. Аудит текущих правил — найденные проблемы

Правила размазаны по 7 местам и уже рассинхронены:

1. Тема захардкожена как AI: news-scout (запросы про LLM/ML), CLAUDE.md «Тематика», consts.ts («AI Digest»), все 4 агента, package.json name. → при смене на crypto прогон продолжит искать AI-новости.
2. Рассинхрон поля обложки (СИСТЕМНЫЙ): пайплайн пишет cover:<URL>, но content.config.ts знает только heroImage (local image()), а layout рендерит heroImage; cover молча отбрасывается Zod. → обложки не отображаются на сайте.
3. Лимиты title/description продублированы в 3 местах (writer.md, page-builder.md, validate-article.js) → рассинхрон при любом изменении.
4. Нет дедупликации: ни один агент не помнит опубликованное. → ежедневный автопрогон повторит темы.
5. `git checkout -B digest/auto` в page-builder сбрасывает ветку на текущий HEAD → потеря накопленных невмёрженных авто-коммитов.
6. Недетерминированные MCP-пины: tavily-mcp@latest, replicate-mcp@alpha.
7. site: 'https://example.com' в astro.config.mjs → ломает RSS/sitemap/canonical.
8. Легаси-мусор: sample-digest-post.md + 3 русские AI-статьи.

## 5. Предлагаемые улучшения

- A. Единый источник правды правил (лимиты + тема) для агентов и хука → устраняет класс багов #3.
- B. Решить #2: дефолт (a) cover URL end-to-end (schema + layout). Альтернатива (b) local heroImage. — TBD.
- C. Дедуп: news-scout читает src/content/blog/ + seen-urls.json, отбрасывает покрытые source-URL/темы.
- D. Фикс ветки: page-builder делает git fetch + checkout существующей digest/auto (без -B reset), коммит поверх.
- E. Запинить MCP-версии; задать ретраи/бэкофф для Tavily/Replicate.
- F. Проставить реальный site; переименовать проект/consts.ts; почистить легаси.

## 6. Предлагаемый стиль (замена секции CLAUDE.md «Стиль»)

```
### Style
- Voice: a CTO of a software company writing to other engineers and
  founders. First-person allowed ("we shipped", "in my experience").
  Opinionated but backed by facts. Conversational, not flat; professional,
  not corporate.
- Language: US-English. Plain English base; Economist-style concision for
  business, Google devdocs conventions for engineering passages.
- Explain trade-offs, not just events: why it matters for building or
  running a product. One concrete takeaway per article.
- Active voice. Vary sentence length for rhythm; avoid run-ons.
- Define every acronym on first use: DeFi, L2, TVL, DAO, zk-rollup, MEV.
- No hype/shilling: no "to the moon", "revolutionary", "game-changer",
  "next-gen", "unlock", "seamless", "web3 magic".
- No AI-markers: "delve", "landscape", "testament to", "in the realm of",
  "tapestry", "underscore", hype-verb "leverage".
- Facts over adjectives. No price predictions, no investment advice.
- Distinguish token vs equity, protocol vs company, mainnet vs testnet,
  custody vs non-custodial.
- Cite the primary source: project post, audit report, on-chain data,
  or regulatory filing.
```

## 7. Черновой план работ (TDD-порядок, исполнение позже)

Порядок: сначала тесты, потом реализация; узкие правки; делегирование по агентам.

Фаза 0 — тестовая база: добавить Vitest в package.json; вынести чистую логику validate-article в модуль; тесты на лимиты/frontmatter. (test-runner + code-writer)
Фаза 1 — правила/контент: обновить CLAUDE.md (Тематика=crypto, Style=CTO-блок); единый источник правды лимитов/темы. (code-writer)
Фаза 2 — обложка (#2, после решения a/b): правка content.config.ts + layout + writer.md + validate-article + page-builder согласованно. (code-writer + test-runner)
Фаза 3 — агенты под crypto/US-English: news-scout (crypto-запросы + дедуп через seen-urls.json + чтение blog), writer (CTO-стиль, лимиты из единого источника), cover-artist (crypto editorial prompt), page-builder (фикс ветки #5, лимиты). (code-writer)
Фаза 4 — инфра: astro.config site=реальный домен; consts.ts=Crypto Digest; переименование проекта crypto-digest; пины MCP; ретраи. (code-writer + git-fs-runner)
Фаза 5 — чистка легаси: удалить sample-digest-post.md + 3 русские AI-статьи. (git-fs-runner)
Фаза 6 — раннер (после решения GitHub Actions/VPS): cron-обвязка, секреты, деплой-хук. (code-writer + git-fs-runner)
Фаза 7 — проверка: build, тесты, ui-tester smoke, curl рендеринга обложки/RSS/sitemap. (test-runner + ui-tester)

## 8. Делегирование (модель-политика)

- Анализ/архитектура/ревью → fable (при недоступности — coordinator сам).
- Написание кода → sonnet (code-writer, type-fixer, refactor-applier).
- FS/git/тесты/логи/доки → haiku (git-fs-runner, test-runner, docs-changelog-writer, ui-tester).
