# 02 — `pattr` / `vattr` / `rattr` _initialize

## Что
Семейство декларативных конструкторов из `attr_extras`. Объявляешь параметры одной строкой, получаешь `initialize` + ридеры. Разница — в видимости и семантике ридеров.

## Реальный код
```ruby
# приватные ридеры (инкапсуляция) — снаружи не видны
class SpentTimeDuration
  pattr_initialize :user_rate
end

# публичные ридеры
class NoTopic < NullObject
  rattr_initialize %i[id linked]
end

# value-семантика: == сравнивает по значению, не по identity
class IncompleteDate
  vattr_initialize :year, :month, :day
end
```

## Как работает
| Хелпер | Ридеры | Семантика | Когда |
|--|--|--|--|
| `pattr_initialize` | приватные | обычная | сервисы, прячем зависимости |
| `rattr_initialize` | публичные | обычная | когда значение нужно снаружи |
| `vattr_initialize` | публичные | `==`/`eql?`/`hash` по значению | value-объекты |
| `method_object` | приватные | + `.call` | callable-объекты (см. [01](01-method-object.md)) |

## Зачем / где полезно
- Ноль бойлерплейта на конструктор + ридеры.
- Имя хелпера сразу говорит намерение: приватный/публичный/по-значению.
- `vattr` бесплатно даёт корректное сравнение — важно для value-объектов в `Set`/ключах хеша.

## Альтернатива
`dry-initializer` (+ типы), `Data.define` (Ruby 3.2+, иммутабельные value-объекты из коробки).
