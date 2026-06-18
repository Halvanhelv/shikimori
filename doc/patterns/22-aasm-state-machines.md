# 22 — `aasm` (конечные автоматы вместо флагов)

## Что
Состояние сущности в одной колонке + чёткие правила переходов. Замена булевым флагам, которые рассинхронизируются.

## Реальный код
```ruby
class Video < ApplicationRecord
  include AASM
  aasm column: 'state', create_scopes: false do
    state Types::Video::State[:uploaded], initial: true
    state Types::Video::State[:confirmed]
    state Types::Video::State[:deleted]

    event :confirm do
      transitions from: [:uploaded, :deleted], to: :confirmed
    end
    event :del do
      transitions from: [:uploaded, :confirmed], to: :deleted
    end
  end
end
```

## Как работает
- Одна колонка `state` вместо `is_confirmed`/`is_deleted` (которые могут противоречить).
- `event :confirm` генерит `video.confirm!` (с сохранением), `video.confirm`, предикат `video.confirmed?`.
- `transitions from:` запрещает невалидные переходы → исключение, а не молчаливая порча.
- Состояния — `dry-types` enum (один источник правды значений).
- Колбэки переходов: `after`, `success`, `guard` (см. [12 — версии](12-versions-moderation.md)).

## Зачем / где полезно
Модерация, заказы (pending→paid→shipped), импорты, загрузки. Любой workflow, где важна валидность переходов. Используется в Video, Poll, AbuseRequest, ListImport, Contest, Version.
