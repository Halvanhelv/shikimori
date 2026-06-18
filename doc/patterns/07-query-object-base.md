# 07 — `QueryObjectBase` (чейн над `ActiveRecord::Relation`)

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
