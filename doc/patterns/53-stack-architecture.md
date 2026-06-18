# 53 — Архитектура стека (4 сервиса)

## Что
Shikimori — не один процесс, а 4 кооперирующих сервиса на разных языках. Каждый закрывает то, что плохо даётся Rails.

## Карта
```
                         ┌─────────────────────────┐
        браузер ◄──ws────┤  faye-server (Node.js)  │  realtime push
            │            └────────────▲────────────┘
            │ HTTP                    │ publish (HTTP + token)
            ▼                         │
   ┌──────────────────────────────────────────────┐
   │           shikimori (Rails monolith)          │
   │  модель, API, очереди (sidekiq), вьюхи         │
   └───┬───────────────────┬──────────────┬────────┘
       │ HTTP POST          │ HTTP GET     │ <img camo-url>
       │ (изменение оценки)  │ (юзер/списки)│
       ▼                    │              ▼
   ┌────────────────────────┴──┐   ┌─────────────────────┐
   │  neko-achievements (Elixir)│   │  camo-server (Node) │
   │  расчёт ачивок, в памяти   │   │  HMAC-прокси картинок│
   └────────────────────────────┘   └─────────────────────┘
```

## Кто за что
| Сервис | Язык | Роль | Почему отдельно |
|--|--|--|--|
| shikimori | Ruby/Rails | домен, API, вьюхи, очереди | ядро |
| neko-achievements | Elixir | расчёт достижений | конкурентность + состояние в памяти |
| faye-server | Node.js | websocket-брокер | масштаб сокетов независимо от Rails |
| camo-server | Node.js | прокси картинок (HMAC) | privacy + mixed-content + CDN-баланс |

## Запуск (Procfile)
```
neko:  cd ../neko-achievements && SHIKIMORI_LOCAL=true mix run --no-halt
camo:  CAMO_KEY=abc PORT=5566 ... coffee ../camo-server/server.coffee
faye:  cd ../faye-server && FAYE_PORT=9292 FAYE_KEY=xxx node server.js
sidekiq: bundle exec sidekiq -C config/sidekiq.yml
```
Overmind держит все процессы; соседние репо лежат рядом (`../neko-achievements` и т.д.).

## Принцип
Монолит остаётся монолитом для домена, но «неудобное для Ruby» вынесено в спец-сервисы: тяжёлый параллельный расчёт → Elixir, тысячи сокетов → Node, проксирование картинок → Node. Связь — простыми HTTP/ws + общий токен.

Детали: [17 — ачивки](17-achievements-neko.md), [24 — faye](24-faye-realtime.md), [23 — camo](23-camo-image-proxy.md), [54 — двунаправленность neko](54-neko-bidirectional.md), [55 — токены между сервисами](55-service-tokens.md).
