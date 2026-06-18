# 59 — Внутренности camo-сервера (безопасный прокси)

## Что
camo-server (Node/CoffeeScript) — не просто «качай и отдавай». Это защищённый прокси: проверка подписи, allowlist хостов, лимиты, security-заголовки, только картинки.

## Реальный код
```coffee
# server.coffee — конфиг защиты
shared_key      = process.env.CAMO_KEY                 # ключ HMAC-подписи URL
allowed_hosts   = (process.env.CAMO_ALLOWED_HOSTS || '').split(',')  # белый список доменов
max_redirects   = process.env.CAMO_MAX_REDIRECTS   || 4
socket_timeout  = process.env.CAMO_SOCKET_TIMEOUT  || 10
content_length_limit = process.env.CAMO_LENGTH_LIMIT || 5242880      # потолок размера

# только эти MIME пропускаются (mime-types.json)
accepted_image_mime_types = JSON.parse(Fs.readFileSync("mime-types.json"))

# жёсткие security-заголовки на каждый ответ
default_security_headers =
  "X-Frame-Options": "deny"
  "X-Content-Type-Options": "nosniff"
  "Content-Security-Policy": "default-src 'none'; img-src data:; style-src 'unsafe-inline'"
  "Strict-Transport-Security": "max-age=31536000; includeSubDomains"
```

## Слои защиты
| Слой | Зачем |
|--|--|
| HMAC-подпись URL (`shared_key`) | нельзя подделать/перебрать чужие URL ([23](23-camo-image-proxy.md)) |
| allowlist хостов | проксируем только с доверенных booru-доменов |
| MIME-allowlist | отдаём только картинки, не HTML/JS/exe |
| `content_length_limit` | защита от гигантских файлов (DoS памяти) |
| `socket_timeout` | висящий апстрим не держит коннект вечно |
| `max_redirects` | не уведут в бесконечную цепочку редиректов |
| security-заголовки | `CSP`, `nosniff`, `X-Frame-Options` — браузер не исполнит контент как код |

## Как работает
- Запрос приходит с подписанным URL → camo проверяет HMAC → если хост в allowlist и подпись валидна, качает оригинал (с таймаутом, лимитом размера, лимитом редиректов).
- Проверяет MIME ответа: не картинка → 404.
- Отдаёт юзеру с жёсткими security-заголовками (контент не может выполниться как скрипт, не встроится во фрейм).

## Зачем / где полезно
Любой прокси внешнего контента обязан быть параноиком: подпись + allowlist + лимиты + MIME-проверка + security-заголовки. Иначе прокси становится SSRF/DoS/XSS-вектором. Учит: «проксировать чужое» = «защищать на каждом слое».
