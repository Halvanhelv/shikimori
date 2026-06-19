# 06 — `ShallowAttributes` value-объекты

## Простыми словами
Иногда нужен лёгкий объект с типизированными полями, но без всей тяжести модели ActiveRecord (без базы, без id). `ShallowAttributes` даёт такой объект: объявил поля с типами — он сам приводит значения к нужному виду. Как компактная карточка-значение «дата выхода аниме», которую можно положить хоть в jsonb-колонку, хоть распарсить из строки.

## Что
Лёгкие value-объекты с типизированными атрибутами и коэрцией. Легче `ActiveModel`, годятся для jsonb-колонок и доменных значений.

## Реальный код
```ruby
class IncompleteDate
  include ShallowAttributes
  include Comparable

  class NilInteger
    def coerce(value, _opts = {}) = value.to_i if value.present?
  end

  attribute :year, NilInteger, allow_nil: true
  attribute :month, NilInteger, allow_nil: true
  attribute :day, NilInteger, allow_nil: true

  # полиморфный конструктор: String / Date / Hash / другой IncompleteDate
  def self.new(object = nil)
    return super({}) if object.blank?
    case object
    when String then d = Date.parse(object); super(year: d.year, month: d.month, day: d.day)
    when Date, Time then super(year: object.year, month: object.month, day: object.day)
    else super(object)
    end
  end
end
```

## Как работает
- `attribute :year, NilInteger` — кастует значение через свою `.coerce`.
- Переопределённый `.new` принимает разные типы → удобно строить из чего угодно.
- `include Comparable` + `<=>` → даты можно сравнивать.

## Зачем / где полезно
Доменные значения с нестандартной семантикой — «2019, месяц неизвестен» (аниме-анонсы), деньги, диапазоны. Хранение в jsonb, парсинг API-пейлоадов. Современная альтернатива: `Data.define` + `ActiveModel::Type`.

## 🆚 Классический Rails
**Обычно:** под такие данные либо заводят полноценную AR-модель (тяжело, тянет базу), либо хранят сырой хеш и парсят/кастуют значения вручную в каждом месте.
**Здесь:** лёгкий value-объект с типизированными атрибутами и автоматической коэрцией — без базы, но с приведением типов и сравнением (`Comparable`).

## Бенефиты
- Типизация и коэрция значений из коробки, без ручного каста.
- Легче ActiveModel — годится для jsonb и доменных значений.
- Полиморфный конструктор: один объект строится из String/Date/Hash.
- Сравнение по значению через `Comparable`.
