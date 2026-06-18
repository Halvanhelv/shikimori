# 42 — Кастомные валидаторы (`EachValidator`)

## Что
Переиспользуемые валидаторы как отдельные классы. Сложная логика (нормализация имён под PostgreSQL, проверка уникальности с учётом регистра/диакритики) вынесена из модели.

## Реальный код
```ruby
class NameValidator < ActiveModel::EachValidator
  FORBIDDEN_NAMES = /\A(?:#{PREDEFINED_PATHS.join '|'}|#{BANNED_NICKNAMES.join '|'})\Z.../mix

  def validate_each(record, attribute, value)
    return unless value.is_a?(String)
    record.errors.add(attribute, :taken) if validate_value(record, value)
    record.errors.add(attribute, :abusive) if Moderations::Banhammer.instance.abusive?(value)
  end

  private

  # уникальность ника/клуба с нормализацией кириллицы и диакритики в SQL
  def postgres_word_normalizer(text)
    <<~SQL.squish
      translate(lower(unaccent(#{text})),
        'абвгдеёзийклмнопрстуфхцьіο0', 'abvgdeezijklmnoprstufxc`ioo')
    SQL
  end
end

# в модели:
validates :nickname, name: true
```

## Как работает
- `ActiveModel::EachValidator` + `validate_each(record, attribute, value)` → подключается как `validates :field, name: true`.
- `FORBIDDEN_NAMES` — зарезервированные пути (`/api`, `/admin`...), чтобы ник не «угнал» роут.
- `postgres_word_normalizer` — нормализует в SQL: `unaccent` + транслит кириллицы → «Аня» и «Anya» считаются занятыми одинаково (анти-обход).

## Зачем / где полезно
Сложные/переиспользуемые правила валидации (имена, URL, эксклюзивные комбинации полей). Выносит логику из модели, тестируется изолированно, вешается декларативно на любое поле.
