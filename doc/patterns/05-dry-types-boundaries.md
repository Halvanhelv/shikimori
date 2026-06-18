# 05 — `dry-types` на границах

## Что
Строгие enum'ы с коэрцией. В отличие от Rails enum (глотает любую строку), `dry-types` бракует чужие значения прямо на входе.

## Реальный код
```ruby
module Types
  include Dry.Types()

  module UserRate
    Status = Types::Strict::Symbol
      .constructor(&:to_sym)                            # строку → симол (граница HTTP)
      .enum(*::UserRate.statuses.keys.map(&:to_sym))    # допустимый набор из БД
  end

  module Ad
    Placement = Types::Strict::Symbol.constructor(&:to_sym).enum(:menu, :content, :footer)
  end
end

# использование:
enumerize :status, in: Types::Anime::Status.values, predicates: true
# мусорное значение → Dry::Types::ConstraintError сразу
```

## Как работает
- `Strict::Symbol` — не приводит молча, бракует не-символ.
- `.constructor(&:to_sym)` — но строку в символ переводит (граница параметров).
- `.enum(...)` — фиксирует набор + даёт `.values` для дропдаунов/`enumerize`.
- Источник значений — сама БД (`UserRate.statuses.keys`), один источник правды.

## Зачем / где полезно
Валидаторы входа (GraphQL/REST), контракты сервисов, парсинг внешних данных. Принцип: **тип как граница**. Мусор ловится рано, с понятной ошибкой, а не всплывает глубоко в запросе.
