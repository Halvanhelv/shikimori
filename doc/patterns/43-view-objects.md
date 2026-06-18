# 43 — ViewObject (vs Decorator)

## Что
Объект представления, не привязанный к одной модели: композиция данных для куска страницы (меню, дашборд, реклама, лейаут). Легче декоратора.

## Реальный код
```ruby
class ViewObjectBase
  include Draper::ViewHelpers   # доступ к h. (хелперы вьюх)
  include Translation
  prepend ActiveCacher.instance # кеш дорогих методов (см. 03)

  def read_attribute_for_serialization(attribute) = send(attribute)
end

# пример: Ad — реклама (≈470 строк): правила показа + рендер баннеров
class Ad < ViewObjectBase
  def html
    return unless AdsPolicy.new(...).allowed?   # гейт через политику
    # сборка баннера по правилам
  end
end
```

## Как работает
- **Decorator** оборачивает **одну модель** (`object`) и добавляет ей презентацию (см. [39](39-decorator-caching.md)).
- **ViewObject** не привязан к модели — собирает представление из нескольких источников (конфиг, политики, запросы).
- `Draper::ViewHelpers` даёт доступ к `h.` (роуты, хелперы) вне вьюхи.
- Кеш дорогих вычислений через `prepend ActiveCacher`.

## Зачем / где полезно
Сложные блоки UI без «своей» модели: топ-меню, дашборд, лейаут, реклама, open graph. Логика представления уезжает из вьюх/хелперов в тестируемый объект. В Rails 8 эту роль закрывает ViewComponent.
