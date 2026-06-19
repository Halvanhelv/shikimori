# 22 — `aasm` (конечные автоматы вместо флагов)

## Простыми словами
Состояние объекта (загружено / подтверждено / удалено) хранится в одной колонке, а переходы между состояниями описаны правилами — нельзя перескочить куда попало. Это как светофор: он переключается строго по схеме, а не «и красный, и зелёный одновременно».

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

## 🆚 Классический Rails
**Обычно:** статус кодируют набором булевых колонок (`is_confirmed`, `is_deleted`) и руками меняют их в коде — флаги рассинхронизируются, появляются невозможные комбинации.
**Здесь:** одна колонка `state` + декларативные `event`/`transitions` через AASM, которые запрещают невалидные переходы и генерят методы `confirm!`, `confirmed?` и т.п.

## Бенефиты
- Невозможных состояний нет — одна колонка вместо противоречивых флагов.
- Недопустимый переход бросает исключение, а не молча портит данные.
- Готовые методы переходов и предикаты из коробки.
- Колбэки (`guard`, `after`, `success`) — единое место для логики перехода.
