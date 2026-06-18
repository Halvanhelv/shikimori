# 28 — `rack-attack` (адаптивный throttle)

## Что
Rate-limiting на уровне Rack с разными лимитами под разных клиентов и safelist'ами для партнёров/внутренних сервисов.

## Реальный код
```ruby
# реальный IP из-за прокси
module Rack::Attack::Request::RealIpFix
  def real_ip
    (env['HTTP_X_FORWARDED_FOR'].presence || env['HTTP_X_REAL_IP'].presence || ip)&.split(',')&.first
  end
end

# партнёр-скрапер получает больший лимит
Rack::Attack.throttle('per second', limit: 15 * MODIFIER, period: 1.second) do |req|
  req.real_ip if req.user_agent == SMOTRET_ANIME_USER_AGENT
end
Rack::Attack.throttle('req/ip', limit: 5 * MODIFIER, period: 1.second) do |req|
  req.real_ip if req.user_agent != SMOTRET_ANIME_USER_AGENT
end

# safelist: внутренний neko, загрузки скриншотов, dev
Rack::Attack.safelist(...) { ... }
```

## Как работает
- Лимит зависит от `user_agent`: известный партнёр — 15 req/s, остальные — 5.
- `real_ip` вытаскивается из заголовков прокси (за CDN/балансером `req.ip` бесполезен).
- Safelist полностью пропускает доверенные источники.

## Зачем / где полезно
Защита от DDoS/перебора, дифференцированные лимиты (партнёр vs аноним), обход для внутренних сервисов. Учит: throttle не «один на всех», а адаптивный по контексту запроса.
