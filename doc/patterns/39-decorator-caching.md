# 39 — Кеширование на границе декоратора

## Что
Декораторы (Draper) — слой презентации, где агрессивно кешируются дорогие запросы, чтобы вьюха дёргала их много раз бесплатно.

## Реальный код
```ruby
class AniMangaDecorator < DbEntryDecorator
  prepend ActiveCacher.instance
  instance_cache :critiques_count, :reviews_count, :current_rate, :chronology,
    :roles, :related, :friend_rates, :external_links, :topic_views

  def chronology = Animes::ChronologyQuery.new(object).fetch   # дорогой запрос
  def roles      = RolesQuery.new(object)
end
```

## Как работает
- Декоратор оборачивает модель (`object`) и добавляет презентационную логику + хелперы (`Draper::ViewHelpers`).
- Дорогие методы кешируются через [instance_cache](03-instance-cache-prepend.md): первый вызов считает, дальше — из ivar.
- Запросы инкапсулированы в query-объекты ([07](07-query-object-base.md)), декоратор их склеивает.
- Граница: модель — данные, query — запросы, декоратор — представление + кеш.

## Зачем / где полезно
Страницы с десятками вычисляемых блоков (карточка аниме: роли, похожее, хронология, оценки друзей). Кеш на границе вьюхи → один рендер не бьёт по БД повторно за тем же. Альтернатива в Rails 8 — ViewComponent + явная мемоизация.
