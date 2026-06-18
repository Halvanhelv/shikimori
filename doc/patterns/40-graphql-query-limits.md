# 40 — GraphQL: лимиты глубины/сложности

## Что
Защита GraphQL-API от тяжёлых/злонамеренных запросов: ограничение глубины и «стоимости» запроса + нормализация типов на входе.

## Реальный код
```ruby
class ShikimoriSchema < GraphQL::Schema
  query Types::QueryType
  mutation Types::MutationType

  query_analyzer LogQueryDepth
  query_analyzer LogQueryComplexityAnalyzer

  max_depth 5          # глубже 5 уровней вложенности — отказ
  max_complexity 190   # суммарная «стоимость» полей — потолок

  class << self
    def execute(query, *, **)
      super(normalize_positive_integer_types(query), *, **)
    end

    def normalize_positive_integer_types(query)
      query.gsub('$page: Int', '$page: PositiveInt')   # подмена типа на входе
           .gsub('$limit: Int', '$limit: PositiveInt')
    end
  end
end
```

## Как работает
- `max_depth` — режет глубоко вложенные запросы (защита от рекурсивных bomb-запросов).
- `max_complexity` — каждое поле стоит «очки»; сумма выше лимита → отказ. Не дать одним запросом выгрести всё.
- `query_analyzer` — логирует фактическую глубину/сложность (мониторинг/тюнинг лимитов).
- `execute` переопределён → нормализует типы скаляров (`Int` → `PositiveInt`) до выполнения.

## Зачем / где полезно
Любой публичный GraphQL: без лимитов один запрос может положить БД. Учит: GraphQL надо явно ограничивать по depth/complexity, а не доверять клиенту. Аналог для REST — пагинация + rate-limit ([28](28-rack-attack.md)).
