# 21 — `redis-mutex` (распределённые локи)

## Что
Лок между процессами/серверами (обычный `Mutex` работает только внутри процесса). Сериализует критические секции.

## Реальный код
```ruby
# ленивое создание топика под гонкой — два коммента не создадут две темы
RedisMutex.with_lock(mutex_key, block: 30.seconds, expire: 30.seconds) do
  apply_commentable comment
end
def mutex_key = "comment_#{@params[:commentable_id]}_#{@params[:commentable_type]}"

# block: 0 → не ждём; занято → бросаем (и повторяем позже)
RedisMutex.with_lock("achievements/track_#{user_id}", block: 0, expire: 60) { ... }

# эксклюзивный импорт: второй запуск молча выходит
RedisMutex.with_lock('import_toshokan_torrents', block: 0) { ... }
```

## Как работает
- `block:` — сколько ждать лок (`0` = не ждать → `RedisMutex::LockError`; `30.seconds` = ждать).
- `expire:` — автоосвобождение (страховка от зависшего процесса).
- Ключ = семантика секции. Для комментов — пара `commentable_id+type`: к одному топику сериализуем, к разным — параллельно.

## Зачем / где полезно
- Идемпотентность импортов/синков (один воркер качает).
- `find_or_create` под гонкой (ленивое создание ресурса).
- Внешний API с лимитом одного коннекта.

Используется в 8 местах проекта. См. отличие от [19 — unique-jobs](19-sidekiq-unique-jobs.md).
