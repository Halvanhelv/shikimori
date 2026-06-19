# 07 — `QueryObjectBase` (чейн над `ActiveRecord::Relation`)

## Простыми словами
Сложные запросы к базе (фильтры, сортировки, пагинация) выносят из модели в отдельный объект, который можно собирать по кусочкам цепочкой. Каждый шаг возвращает новый объект, не ломая исходный. Как конструктор LEGO: добавляешь деталь — получаешь новую сборку, а старая остаётся целой.

## Что
Базовый класс для query-объектов. Оборачивает relation так, что каждый метод-запрос возвращает **новый** query-объект (иммутабельная цепочка).

## Реальный код
```ruby
class QueryObjectBase
  prepend ActiveCacher.instance
  QUERY_METHODS = %i[joins includes preload where order limit offset none except] +
    (defined?(ArLazyPreload) ? %i[lazy_preload] : [])
  vattr_initialize :scope

  QUERY_METHODS.each do |m|
    define_method(m) { |*args| chain @scope.public_send(m, *args) }
  end

  def paginate(page, limit, offset = 0)
    chain PaginatedCollection.new(@scope.offset(offset + limit * (page - 1)).limit(limit), page, limit)
  end

  def lazy_map(&block) = chain TransformedCollection.new(@scope, :map, block)
  def method_missing(m, *, &) = @scope.send(m, *, &)   # прозрачный фолбэк
  private def chain(scope) = self.class.new(scope)      # КЛЮЧ: новый объект
end
```

## Как работает
- `QUERY_METHODS` метапрограммно делегируются в `@scope` и оборачиваются назад через `chain`.
- `chain = self.class.new(scope)` — иммутабельность: исходный объект не мутируется.
- `method_missing` пробрасывает всё неизвестное в relation → доступны любые скоупы модели.
- `lazy_map`/`lazy_filter` — отложенные трансформации, не материализуют relation сразу.

## Зачем / где полезно
Сложная логика фильтрации/поиска/пагинации вне модели. Тестируется изолированно, композируется. Альтернатива жирным scope и `ransack`. Конкретное применение — см. [15 — DSL фильтров](15-animes-query-dsl.md).

## 🆚 Классический Rails
**Обычно:** сложные выборки делают через `scope` в самой модели или через цепочки `where`/`order` прямо в контроллере. Модель пухнет от десятков скоупов, а логика поиска размазана по контроллерам.
**Здесь:** логика запроса вынесена в отдельный query-объект с иммутабельной цепочкой (`chain` создаёт новый объект), а `method_missing` всё равно даёт доступ ко всем скоупам модели.

## Бенефиты
- Модель не пухнет от скоупов — логика запроса в отдельном классе.
- Иммутабельность: исходный объект не мутируется, цепочки безопасны.
- Тестируется и композируется изолированно от контроллера.
- Ленивые трансформации (`lazy_map`) не материализуют relation раньше времени.
