# 17 — Ачивки через внешний Elixir-сервис

## Простыми словами
Подсчёт достижений («посмотрел 100 аниме» и т.п.) — тяжёлая работа, поэтому её отдали отдельной программе на языке Elixir, который хорошо с этим справляется. Rails тут как официант: принимает заказ, относит на кухню (Elixir), приносит готовое блюдо клиенту, но сам не готовит.

## Что
Логика правил достижений вынесена из Rails в отдельный сервис на Elixir (`neko-achievements`). Rails только триггерит расчёт и сохраняет результат.

## Реальный код
```ruby
# ТРИГГЕР: юзер меняет список → фоновая задача
Achievements::Track.perform_async(@resource.user_id, @resource.id, Types::Neko::Action[:put])

# ВОРКЕР: под mutex'ом по юзеру + retry на сбоях сети
class Achievements::Track
  def perform(user_id, user_rate_id, action)
    RedisMutex.with_lock("achievements/track_#{user_id}", block: 0, expire: 60) do
      Neko::Update.call(User.find(user_id), user_rate_id:, action: Types::Neko::Action[action])
    end
  rescue *NET_ERRORS then self.class.perform_in(1.minute, ...)   # Elixir упал → повтор
  rescue RedisMutex::LockError then self.class.perform_in(5.seconds, ...)
  end
end

# ЗАПРОС: HTTP в Elixir, тот считает правила, отдаёт дельту
Neko::Request.call(params)  # → { added:, updated:, removed: }
```

## Как работает
1. Изменил оценку → `perform_async` (мгновенный ответ юзеру).
2. Воркер собирает данные оценки, POST'ит в Elixir (`localhost:4000/user_rate`).
3. Elixir прогоняет все правила, возвращает что добавить/обновить/убрать.
4. `Neko::Apply` пишет в таблицу `achievements` + пушит уведомление через Faye.
5. Сами правила — в YAML, описание берётся из `Neko::Rule` (см. [11](11-method-missing-delegation.md)).

## Зачем / где полезно
Когда тяжёлый параллельный расчёт лучше отдать специализированному рантайму (Elixir/Go), оставив Rails тонким посредником. Учит: триггер→фон→внешний мозг→сохранение→push. Граница через HTTP + dry-types на типах действий.

## 🆚 Классический Rails
**Обычно:** всю бизнес-логику (включая правила достижений) держат внутри Rails — в моделях, сервисах и колбэках, исполняя её тем же ruby-процессом.
**Здесь:** правила вынесены в отдельный Elixir-сервис, Rails только триггерит расчёт фоновой задачей, ходит в него по HTTP и сохраняет присланную дельту.

## Бенефиты
- Тяжёлый параллельный расчёт идёт на рантайме, заточенном под это.
- Rails остаётся тонким и не нагружается логикой правил.
- Сервисы масштабируются и деплоятся независимо.
- Чёткая граница через HTTP + dry-types защищает от рассинхрона типов.
