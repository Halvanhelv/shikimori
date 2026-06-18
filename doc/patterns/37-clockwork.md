# 37 — `clockwork` (планировщик в процессе)

## Что
Лёгкий планировщик задач в Ruby-процессе вместо системного cron. Расписание — кодом, с точным временем.

## Реальный код
```ruby
# config/clock.rb
every(5.minutes, 'pghero.query_stats') { PgHero.capture_query_stats }

every(30.minutes, 'half-hourly.import', at: ['**:15', '**:45']) do
  MalParsers::FetchPage.perform_async('anime', 'updated_at', 0, 3)
end

every(1.day, 'daily.imports', at: '22:30') do
  # ночные задачи
end
```

## Как работает
- `every(interval, name, at:)` — задаёт периодичность и точное время.
- Сам ничего тяжёлого не делает — только `perform_async` в Sidekiq (планировщик отделён от исполнения).
- Один процесс держит расписание; задачи уходят в очереди.

## Зачем / где полезно
Периодические задачи (импорты, статистика, чистки) когда хочется расписание в репозитории, а не в crontab сервера. Точное время (`at: '22:30'`), читаемо, версионируется. Аналоги: `sidekiq-cron`, `whenever`, Solid Queue recurring.
