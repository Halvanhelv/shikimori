# 04 — `DslAttribute` (конфиг на уровне класса)

## Простыми словами
Иногда подклассы отличаются не поведением, а парой настроек: «работаю с моделью Review», «меня можно публиковать». Вместо переопределения методов их задают коротким объявлением прямо в теле класса. Как настройки в шапке документа: «шрифт — такой, поля — такие», а не переписывать весь текст под себя.

## Что
Свой модуль (`lib/dsl_attribute.rb`). Даёт декларативную настройку подкласса вместо переопределения методов. Значение хранится и как ivar класса, и как константа.

## Реальный код
```ruby
class UserContent::CreateBase
  extend DslAttribute
  dsl_attribute :klass
  dsl_attribute :is_publishable
end

class Review::Create < UserContent::CreateBase
  klass Review            # настройка подкласса декларацией
  is_publishable true
end
```

Определение:
```ruby
module DslAttribute
  def dsl_attribute(attribute_name, default_value = nil)
    define_singleton_method(attribute_name) do |value|
      instance_variable_set("@#{attribute_name}", value)
      const_set(attribute_name.to_s.upcase, value)
    end
    define_method(attribute_name) do
      self.class.instance_variable_get("@#{attribute_name}") || default_value
    end
  end
end
```

## Как работает
- `dsl_attribute :klass` создаёт класс-метод `klass(value)` (сеттер) и инстанс-метод `klass` (геттер).
- Подкласс вызывает `klass Review` в теле → настройка читаема и наследуема.

## Зачем / где полезно
Базовые классы операций/политик с «параметрами поведения». Декларативно, без церемоний Rails-DSL. Альтернатива: `class_attribute` из Rails (проще, но без const).

## 🆚 Классический Rails
**Обычно:** настройку подкласса делают через переопределение метода (`def klass = Review`) или через `class_attribute :klass` с присваиванием `self.klass = Review`. Чистого «декларативного» синтаксиса в самом теле класса по умолчанию нет.
**Здесь:** свой мини-DSL `klass Review` читается как декларация, а значение дополнительно кладётся в константу (`KLASS`) — удобно для ссылок из других мест.

## Бенефиты
- Подкласс настраивается одной читаемой строкой, а не методом.
- Значение доступно и как метод, и как константа.
- Настройка наследуется и легко переопределяется в потомках.
