# 24 — Faye: realtime через websockets

## Простыми словами
Новые комментарии и уведомления появляются на странице сами, без перезагрузки — сервер сам «толкает» их в браузер по открытому каналу. Это как групповой чат: написал один — сообщение тут же всплыло у всех, кто в этой беседе.

## Что
Комменты, ачивки, обновления прилетают в браузер живьём. Брокер сообщений вынесен в отдельный Node-сервис (`faye-server`), масштабируется независимо от Rails.

## Реальный код
```ruby
# один коммент пушится сразу в несколько каналов
def comment_channels(comment, channels)
  channels +
    linked_channels(comment.commentable) +
    comment.faye_channels +
    ["/#{comment.commentable_type.downcase}-#{comment.commentable_id}", "/forum-#{topic.forum_id}"]
end

# реактор EventMachine стартует лениво в фоновом потоке
def run_event_machine
  unless EM.reactor_running?
    Thread.new { EM.epoll; EM.set_descriptor_table_size(100_000); EM.run }
    Thread.pass until EM.reactor_running?
  end
end
```

Аутентификация публикаций по общему токену (Node-сторона):
```javascript
if (message.data && message.data.token == key) { delete message.data.token; }
else { message.error = 'No client posting is allowed'; }
```

## Как работает
- Rails публикует событие в Faye через HTTP с токеном; Faye рассылает подписчикам по websocket.
- Канальная маршрутизация: один коммент → `/topic-{id}`, `/forum-{id}`, тред коммента — все подписчики уведомлены.
- Брокер отдельный → веб-сокеты не нагружают Rails-процессы.

## Зачем / где полезно
Живые ленты, уведомления, совместное редактирование. Учит: выноси push-брокер из монолита, маршрутизируй по каналам. Современные аналоги — Action Cable, AnyCable.

## 🆚 Классический Rails
**Обычно:** realtime делают через Action Cable внутри того же Rails-приложения — websocket-соединения висят на тех же процессах, что и обычные запросы.
**Здесь:** брокер сообщений вынесен в отдельный Node-сервис (faye-server); Rails лишь публикует событие по HTTP с токеном, а рассылку по websocket держит независимый процесс.

## Бенефиты
- Долгие websocket-соединения не нагружают Rails-процессы.
- Брокер масштабируется и деплоится отдельно от монолита.
- Канальная маршрутизация: один коммент летит сразу в несколько каналов.
- Простая аутентификация публикаций общим токеном.
