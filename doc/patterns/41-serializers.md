# 41 — Сериализаторы (слой JSON-вывода)

## Что
Слой `ActiveModel::Serializer`: форма JSON для API описывается декларативно, с наследованием (базовый → профильный) и переопределением полей.

## Реальный код
```ruby
class AnimeProfileSerializer < AnimeSerializer   # расширяет базовый
  attributes :rating, :english, :japanese, :score, :description,
    :favoured, :anons, :ongoing, :topic_id, :rates_scores_stats

  has_many :genres
  has_many :studios
  has_one :user_rate, serializer: UserRateFullSerializer

  def user_rate = object.current_rate          # переопределение поля
  def description = object.description.text
  def favoured = object.favoured?

  def rates_scores_stats
    (object.stats&.scores_stats || []).map { |e| { name: e[0].to_i, value: e[1] } }
  end
end
```

## Как работает
- `attributes` — какие поля попадают в JSON.
- Метод с именем поля переопределяет значение (`description` → `.text`, а не сам объект).
- `has_many`/`has_one` — вложенные ассоциации, можно указать свой сериализатор.
- **Наследование**: `AnimeSerializer` (краткий) → `AnimeProfileSerializer` (полный). Не дублируешь общие поля.

## Зачем / где полезно
Стабильная, версионируемая форма API-ответа. Разделение «краткий список» vs «полная карточка» через наследование. Логика формы — в одном месте, тестируется. Принцип тот же, что я применил в gnosis-PR (вынос инлайн-сериализации в отдельный класс).
