# 03 — `instance_cache` через `prepend`

## Простыми словами
Если метод считает что-то долгое (например, лезет в базу), не хочется делать это заново при каждом вызове — лучше посчитать один раз и запомнить. Этот механизм позволяет объявить «закешируй вот эти методы» одной строкой сверху класса, не засоряя сами методы. Как заметка на холодильнике: посмотрел цену один раз, записал, дальше не перепроверяешь каждый раз.

## Что
Свой механизм (`lib/active_cacher.rb`): мемоизация результата метода декларацией наверху класса, без `@x ||=` в теле. Есть две формы: `instance_cache` (в ivar) и `rails_cache` (ещё и в `Rails.cache`).

## Реальный код
```ruby
class AniMangaDecorator < DbEntryDecorator
  prepend ActiveCacher.instance
  instance_cache :critiques_count, :roles, :chronology, :external_links
  rails_cache :some_expensive_html

  def critiques_count = object.critiques.count   # выполнится один раз
end
```

Механизм:
```ruby
target.define_singleton_method(:instance_cache) do |*methods|
  methods.each do |method|
    escaped = method.to_s.include?('?') ? "is_#{method[0..-2]}" : method
    cacher.define_method(method) do |*args|
      value = instance_variable_get("@__#{escaped}")
      value.nil? ? instance_variable_set("@__#{escaped}", prepare_for_cache(super(*args))) : value
    end
  end
end
```

## Как работает
- `instance_cache :foo` определяет `foo` в **prepend'нутом** модуле. Он зовёт твой настоящий `foo` через `super` один раз, кладёт в `@__foo`.
- `prepend` ставит обёртку **перед** классом в цепочке предков → перехват без замусоривания тела метода.
- `?` в имени → ivar `@__is_foo` (нельзя `@__foo?`).
- `prepare_for_cache` конвертит `ActiveRecord::Relation` в массив (иначе кешировали бы ленивый запрос).
- Препендится **клон** модуля (`.instance`), иначе методы пересекутся между классами.

## Зачем / где полезно
Декораторы/презентеры с десятком дорогих вычисляемых свойств, дёргаемых много раз за рендер. Декларация видна сразу, тело методов чистое. `rails_cache` — когда нужно пережить запрос.

## 🆚 Классический Rails
**Обычно:** мемоизацию пишут руками в каждом методе — `def roles; @roles ||= object.roles; end`. Это размазывает кеш-логику по телу метода и плохо работает с `nil`/`false` (паттерн `||=` их не кеширует, см. [90](90-memoization-caveat.md)).
**Здесь:** методы пишутся чисто (`def roles = object.roles`), а кеширование объявляется отдельной строкой `instance_cache :roles`. Обёртка через `prepend` перехватывает вызов и кеширует, не трогая исходный код метода.

## Бенефиты
- Тело методов чистое — видно логику, а не служебный `@x ||=`.
- Список кешируемого собран в одном месте сверху класса.
- `rails_cache` даёт тот же синтаксис, но кеш переживает запрос (общий `Rails.cache`).
- Корректно кеширует `nil`/`false`, в отличие от наивного `||=`.
