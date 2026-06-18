# 19 — `sidekiq-unique-jobs` (дедуп задач)

## Что
Не ставит в очередь дубликат задачи, пока такая же выполняется. Защищает от повторных дорогих расчётов.

## Реальный код
```ruby
class SimilarUsersWorker
  include Sidekiq::Worker
  sidekiq_options(
    lock: :until_executed,                       # лок держится до конца выполнения
    lock_args_method: ->(args) { args.first },   # уникальность по первому аргументу (user_id)
    queue: :cpu_intensive,
    retry: false
  )

  def perform(user_id, type, threshold, cache_key)
    Rails.cache.fetch(cache_key, expires_in: 2.weeks) { fetch(...) }
  end
end
```

## Как работает
- `lock: :until_executed` — пока задача в очереди/работе, дубликат с тем же ключом не встанет.
- `lock_args_method` — что считать «тем же»: тут только `user_id` (прочие аргументы для уникальности игнорятся).
- Юзер кликнул 5 раз → 1 расчёт, не 5.

Режимы: `until_executing` (лок до старта), `while_executing` (мьютекс на исполнение), `until_executed`.

## Зачем / где полезно
Дорогие пересчёты по триггеру юзера, дебаунс вебхуков, «один дайджест на шквал событий». В Solid Queue аналог — `limits_concurrency`.

## Связано
[21 — redis-mutex](21-redis-mutex.md) сериализует на уровне **исполнения**; unique-jobs дедупит на уровне **очереди**. Разные слои.
