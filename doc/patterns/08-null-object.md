# 08 — Null Object

## Что
Объект-«пустышка» вместо `nil`: не падает на вызовах, убирает nil-проверки из вьюх и декораторов.

## Реальный код
```ruby
class NullObject
  prepend ActiveCacher.instance
  instance_cache :base_object

  def respond_to_missing?(name, include_private = false)
    base_object.respond_to?(name) || super
  end
  def method_missing(name, *_args)
    return false if name.to_s.end_with?('?')   # предикат → false
    nil                                         # остальное → nil
  end
  def nil? = true                               # притворяется nil'ом
  private def base_object = base_klass.new
end

class NoTopic < NullObject
  rattr_initialize %i[id linked]
  def comments = Comment.none      # точечно: должен быть relation, не nil
  def comments_count = 0
  private def base_klass = Topic
end
```

## Как работает
- `method_missing`: предикат → `false`, прочее → `nil`.
- `nil? => true` + `respond_to_missing?` через пустой инстанс реального класса → подмена прозрачна для Rails-хелперов.
- Потомок переопределяет только то, где `nil` ломает (например `comments` обязан быть relation, иначе `.each` упадёт).

## Зачем / где полезно
Гостевой юзер (`NullUser` с дефолтными правами), отсутствующие настройки, разорванные связи, дефолтная стратегия. Чистит код от россыпи `&.` и `if x`.

## Когда не надо
Если nil-кейс уже обработан коротко (`x&.foo` или дефолт через `||`), отдельный класс — churn.
