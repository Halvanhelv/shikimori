# 48 — GraphQL-резолвер: lookahead + lazy_preload

## Простыми словами
Резолвер — это код, который отвечает на GraphQL-запрос аниме. Здесь он переиспользует ту же логику фильтрации, что и REST (не дублирует), и умеет «заглянуть вперёд» (lookahead) — посмотреть, какие именно поля попросил клиент, и грузить тяжёлые связи только если они реально нужны. Это как официант, который сначала читает весь заказ целиком, а потом идёт на кухню один раз за всем нужным — а не бегает за каждым блюдом отдельно.

## Что
Резолвер аниме переиспользует тот же `Animes::Query`, что и REST, объявляет аргументы типизированными скалярами, и грузит ассоциации лениво — плюс смотрит «вперёд» (lookahead), что именно запросил клиент.

## Реальный код
```ruby
class Queries::AnimesQuery < Queries::BaseQuery
  type [Types::AnimeType], null: false
  extras [:lookahead]                       # доступ к дереву запроса клиента

  argument :season, Types::Scalars::SeasonString, required: false   # типизированный скаляр
  argument :limit, Types::Scalars::PositiveInt, default_value: 2, description: "Maximum #{LIMIT}"

  def resolve(page:, limit:, lookahead:, season: nil, genre: nil, **rest)
    collection = Animes::Query.fetch(
      scope: Anime.lazy_preload(*PRELOADS),   # авто-фикс N+1 (см. 36)
      params: { page:, season:, genre_v2: genre, **rest },
      user: current_user
    ).paginate(page, limit.to_i.clamp(1, LIMIT)).to_a

    fetch_user_rates(collection) if lookahead.selects?(:user_rate) && current_user  # грузим только если просили
    collection
  end
end
```

## Как работает
- **Один источник фильтрации**: GraphQL и REST зовут общий `Animes::Query` ([15](15-animes-query-dsl.md)) — логика не дублируется.
- **`extras [:lookahead]`**: резолвер видит, какие поля запросил клиент. `lookahead.selects?(:user_rate)` → грузим оценки только если они нужны.
- **`lazy_preload`**: ассоциации подгружаются лениво ([36](36-ar-lazy-preload.md)) — нет N+1 при произвольном наборе полей.
- **Типизированные скаляры-аргументы** (`SeasonString`, `PositiveInt`): валидация формата на входе, как dry-types ([05](05-dry-types-boundaries.md)).
- `limit.clamp(1, LIMIT)` — потолок на размер выборки.

## Зачем / где полезно
GraphQL-резолвер, который не дублирует бизнес-логику и не плодит N+1. Учит: переиспользуй query-объекты между API, используй lookahead для условной загрузки, клампи лимиты.

## 🆚 Классический Rails
**Обычно:** в REST-контроллере набор полей и предзагрузка ассоциаций (`includes`) фиксированы в коде эндпоинта — заранее известно, что грузить. А логику фильтрации нередко дублируют между REST- и GraphQL-частями.
**Здесь:** GraphQL отдаёт выбор полей клиенту, поэтому резолвер смотрит вперёд через `extras [:lookahead]` и грузит тяжёлые связи (`user_rate`) только если их запросили; ассоциации тянутся через `lazy_preload` (авто-фикс N+1), а фильтрация — через общий с REST `Animes::Query`.

## Бенефиты
- Одна логика фильтрации на REST и GraphQL — нет дублирования.
- Тяжёлые связи грузятся только когда клиент их реально запросил.
- `lazy_preload` снимает N+1 при любом наборе запрошенных полей.
- Типизированные скаляры-аргументы и `clamp` ограничивают мусор и размер выборки на входе.
