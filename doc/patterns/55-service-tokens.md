# 55 — Аутентификация между сервисами (токены + HMAC)

## Что
Каждая связь между сервисами защищена простым общим секретом — у каждого свой механизм под свою угрозу.

## Три механизма
### 1. Faye — токен в payload публикации
Только сервер с токеном может публиковать в каналы; клиенты-браузеры только слушают.
```javascript
// faye-server/server.js
if (message.data && message.data.token == key) {
  delete message.data.token;          // токен валиден → пропускаем, вырезаем из payload
} else {
  message.error = 'No client posting is allowed';   // нет токена → запрет публикации
}
```
```ruby
# Rails при публикации добавляет токен
faye_client.publish(channel, data.merge(token: Rails.application.secrets.faye[:token]))
```

### 2. Neko — токен в заголовке + Plug-аутентификация
```elixir
# router.ex
@token "foo"
plug Neko.Plug.Authenticate, token: @token   # отбивает запросы без токена
```
```ruby
# Rails
req.headers['Authorization'] = 'foo'
```

### 3. Camo — HMAC-подпись URL
URL картинки подписан ключом → нельзя подделать или перебрать чужие (см. [23](23-camo-image-proxy.md)).
```ruby
OpenSSL::HMAC.hexdigest('sha1', CAMO_KEY, url)
```

## Как работает / почему по-разному
| Связь | Угроза | Защита |
|--|--|--|
| Rails → Faye | чужой клиент шлёт фейк-события | shared token в payload, вырезается на входе |
| Rails → Neko | посторонний дёргает расчёт | token в `Authorization` + Plug |
| браузер → Camo | подделка/перебор URL картинок | HMAC-подпись URL |

## Зачем / где полезно
Внутренняя межсервисная связь не обязана быть OAuth — часто хватает общего секрета, но **механизм выбирается под угрозу**: токен для «только сервер пишет», HMAC для «URL нельзя подделать», заголовок+middleware для «только свои дёргают эндпоинт».

⚠️ В dev-конфиге секреты примитивные (`token: 'foo'`, `FAYE_KEY=xxx`) — в проде должны быть из секретов/ENV. Учебный момент: не коммить реальные ключи, разделяй dev/prod секреты.
