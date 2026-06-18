# 31 — `enumerize` (типизированные enum)

## Что
Enum'ы с предикатами и i18n поверх атрибута, со значениями из `dry-types`.

## Реальный код
```ruby
class Anime < ApplicationRecord
  enumerize :status, in: Types::Anime::Status.values, predicates: true
  enumerize :rating, in: Types::Anime::Rating.values, predicates: true
  enumerize :kind,   in: Types::Anime::Kind.values,   predicates: true
end

# даёт предикаты:
anime.ongoing?   # true/false
anime.tv?
```

## Как работает
- `in:` — допустимый набор (берётся из `dry-types` enum, см. [05](05-dry-types-boundaries.md)).
- `predicates: true` — автогенерит `anime.ongoing?` и т.п.
- Интеграция с i18n — человекочитаемые названия значений.
- В отличие от Rails-enum (целочисленный маппинг), `enumerize` гибче с источниками значений и хелперами.

## Зачем / где полезно
Статусы/типы/рейтинги с предикатами и локализацией. Используется в 50+ моделях проекта. Один источник значений (dry-types) + удобные хелперы в коде и вьюхах.
