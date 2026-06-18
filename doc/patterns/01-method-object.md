# 01 — `method_object` (callable-объекты)

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

## Зачем / где полезно
- Один use-case = один класс. Контроллер становится тонким.
- Шаги сценария — приватные методы, состояние в `@ivar`, не таскаешь параметры.
- Тестируется в изоляции: `Comment::Create.call(...)`.
- Аналоги в экосистеме: `interactor`, `dry-transaction`, service objects.

## Когда не надо
Для одной строки логики класс избыточен — оставь метод в модели/контроллере.
