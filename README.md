# Web3 Digest

Автоматический дайджест новостей крипторынка, блокчейна и регулирования цифровых активов (MiCA, web3).

## Структура

```
src/
├── content/blog/    — Статьи дайджеста (markdown).
├── components/      — Astro-компоненты.
├── layouts/         — Шаблоны страниц.
├── pages/           — Маршруты: главная, блог, RSS.
├── styles/          — Глобальные стили.
└── assets/          — Изображения и обложки.
```

## Формат статьи

Каждая статья — файл `.md` в `src/content/blog/`:

```markdown
---
title: 'Заголовок статьи'
description: 'Краткое описание в 2–3 предложения.'
pubDate: '2026-03-25'
tags: ['mica', 'regulation']
source: 'https://example.com/original-article'
---

Текст статьи. 300–500 слов.
```
