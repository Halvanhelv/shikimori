# 01 — `method_object` (callable-объекты)

## Простыми словами
Берём одно действие («создать комментарий») и оформляем его как отдельный класс с одной кнопкой `call`. Снаружи — `Comment::Create.call(...)`, внутри — все шаги по порядку. Как рецепт в отдельной карточке: не держишь все рецепты в одном огромном файле.

## Что
Хелпер из гема `attr_extras`. Превращает класс в одноразовый объект-действие с единственным публичным методом `call`. Генерит конструктор, приватные ридеры и класс-фасад `.call`.

## Реальный код
```ruby
class Comment::Create
  method_object %i[params! faye! is_conversion is_forced]
  #                 ! = обязательный параметр; без ! = опциональный (nil)

  def call
    comment = Comment.new @params.except(:commentable_id, :commentable_type)
    RedisMutex.with_lock(mutex_key, block: 30.seconds, expire: 30.seconds) do
      apply_commentable comment
    end
    # ... остальные шаги
    comment
  end

  private

  def mutex_key = "comment_#{@params[:commentable_id]}_#{@params[:commentable_type]}"
  def apply_commentable(comment) = # ...
end

# Вызов:
Comment::Create.call(params: p, faye: f, is_forced: true)
```

## Как работает
`method_object %i[params! faye!]` разворачивается примерно в:
```ruby
def initialize(params:, faye:, is_conversion: nil, is_forced: nil)
  @params, @faye, @is_conversion, @is_forced = params, faye, is_conversion, is_forced
end
private attr_reader :params, :faye, :is_conversion, :is_forced
def self.call(...) = new(...).call
```
`!` форсит обязательность → ошибка на входе, а не `nil` где-то в середине.

## 🆚 Классический Rails
**Обычно:** всю логику «создать коммент» суют в контроллер (`def create; ...30 строк...; end`) или в «толстую» модель. Переиспользовать кусок нельзя, тестировать — только через HTTP-запрос.
**Здесь:** действие — отдельный объект. Контроллер становится одной строкой `Comment::Create.call(...)`, логика тестируется без HTTP, переиспользуется из воркера/консоли.

## Бенефиты
- Тонкий контроллер, логика в одном понятном месте.
- Шаги — приватные методы, состояние в `@ivar`, не таскаешь параметры по цепочке.
- Тест в изоляции: `Comment::Create.call(...)`.
- `!` форсит обязательные параметры → ошибка на входе, а не `nil` в середине.

## Когда не надо
Для одной строки логики класс избыточен — оставь метод в модели/контроллере. Аналоги в экосистеме: `interactor`, `dry-transaction`, обычные service objects.
