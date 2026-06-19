# 92 — Procfile + Overmind (запуск всего стека одной командой)

## Простыми словами
Проект — это не один процесс, а несколько (Rails, сборка фронта, Sidekiq, neko, faye, camo). Чтобы не открывать 6 терминалов вручную, все процессы перечислены в одном файле (`Procfile`) и запускаются одной командой через Overmind.

## Реальный код
```procfile
# Procfile
webpack: bin/shakapacker-dev-server
neko:    cd ../neko-achievements && SHIKIMORI_LOCAL=true mix run --no-halt
camo:    CAMO_KEY=abc PORT=5566 ... coffee ../camo-server/server.coffee
faye:    cd ../faye-server && FAYE_PORT=9292 FAYE_KEY=xxx node server.js
sidekiq: bundle exec sidekiq -C config/sidekiq.yml
```
```sh
overmind start                         # поднять всё
OVERMIND_PROCESSES=camo,faye overmind start   # только часть
```

## Как работает
- Каждая строка `Procfile` = имя процесса + команда запуска (со своими ENV).
- `overmind start` поднимает все процессы разом, собирает их логи, даёт подключаться к каждому.
- Можно запустить подмножество (`OVERMIND_PROCESSES=...`).

## 🆚 Классический подход
**Обычно (больно):** разработчик открывает 6 терминалов, в каждом руками вводит команду с нужными переменными, путается, забывает запустить Sidekiq.
**Здесь:** один файл описывает весь стек, одна команда поднимает всё. Новый разработчик стартует проект за минуту.

## Бенефиты
- Один источник правды «как запускается проект».
- Один вход (`overmind start`), общие логи.
- Легко поднять часть стека под конкретную задачу.

## Когда полезно вне shikimori
Любой проект из нескольких процессов (фронт + бэк + воркер + БД). `Procfile` — стандарт (Heroku, foreman, overmind). Для совсем сложного — docker-compose.
