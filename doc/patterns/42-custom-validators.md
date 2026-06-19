# 42 — Кастомные валидаторы (`EachValidator`)

## Простыми словами
Вместо того чтобы писать сложную проверку («ник не занят, не запрещён, не похож на чужой обходом через транслит») прямо в модели, её выносят в отдельный класс-валидатор. Потом вешают на любое поле одной строкой `validates :nickname, name: true`. Это как готовый «датчик качества»: собрал один раз — и ставь куда нужно, не переписывая логику.

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

## 🆚 Классический Rails
**Обычно:** простые правила вешают встроенными хелперами (`validates :nickname, presence: true, uniqueness: true`), а сложную логику пишут методом `validate :custom_check` прямо в модели — он копится и привязан к одной модели.
**Здесь:** сложная переиспользуемая логика вынесена в отдельный класс `ActiveModel::EachValidator`, который подключается декларативно (`validates :nickname, name: true`) к любой модели. Внутри — нетривиальная нормализация кириллицы/диакритики в SQL для анти-обхода.

## Бенефиты
- Правило пишется один раз и вешается на любое поле любой модели.
- Логика валидации вне модели — модель остаётся тонкой.
- Тестируется изолированно, без подъёма всей модели.
- Сложные кейсы (транслит, диакритика, занятые роуты) спрятаны за простым `name: true`.
