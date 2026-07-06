# Git

- Ветка по умолчанию `master`.
- С фазы 08 PreToolUse-хук блокирует прямой push в `master`. Автоматика коммитит в ветку `digest/auto`, слияние в `master` — вручную по желанию.
- Запрещены `git push --force`, `git reset --hard`, `git commit --amend`.
