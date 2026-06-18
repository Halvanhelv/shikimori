# 15 — DSL фильтров каталога

## Что
URL вида `/animes/season/summer_2024/genre/comedy` превращается в SQL через цепочку фильтр-объектов. Каждый фильтр — отдельный объект, возвращает новый scope.

## Реальный код
```ruby
# app/query_objects/animes/query.rb
new(scope)
  .by_franchise(params[:franchise])
  .by_genre(params[:genre])
  .by_season(params[:season])
  .by_duration(params[:duration])
  # 20+ фильтров чейнятся

# парсинг сезона регэкспами:
case season
  when /^(?<season>[a-z]+)_(?<year>\d{4})$/ then season_sql(...)   # summer_2024
  when /^(?<decade>\d{3})x$/                then decade_sql(...)   # 200x
  when /^(?<from>\d{4})_(?<to>\d{4})$/      then years_sql(...)    # 2020_2025
  when /^\d{4}$/                            then year_sql(...)     # 2024
  else raise InvalidParameterError.new(:season, season)
end
```

## Как работает
- Поверх [QueryObjectBase](07-query-object-base.md): каждый `.by_X` применяет фильтр и возвращает query-объект.
- Фильтры используют `dry-types` для коэрции (см. [05](05-dry-types-boundaries.md)), ошибку перевыбрасывают с именем поля.
- Поддержка отрицания: `!23,!24` → исключить жанры.
- Массивы PostgreSQL через оператор `&&` (пересечение) для жанров.

## Зачем / где полезно
Поиск/фильтры/отчёты с множеством опциональных условий. Нет лапши `if params[...]`, каждый фильтр тестируется отдельно, легко добавить новый. Альтернатива: `ransack` (но он магичнее и хуже контролируется).
