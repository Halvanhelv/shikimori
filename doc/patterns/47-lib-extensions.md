# 47 — Утилиты в `lib/` (DeepStruct и др.)

## Что
Маленькие переиспользуемые утилиты вне Rails-конвенций: `DeepStruct`, `OrderedSet`, расширения `String`, `NamedLogger`.

## Реальный код
```ruby
# lib/deep_struct.rb — OpenStruct, рекурсивно оборачивающий вложенные хеши
class DeepStruct < OpenStruct
  def initialize(hash = nil)
    super
    @table = {}; @hash_table = {}
    hash&.each do |k, v|
      @table[k.to_sym] = v.is_a?(Hash) ? self.class.new(v) : v   # рекурсия по вложенным
      @hash_table[k.to_sym] = v
      new_ostruct_member!(k)
    end
  end

  def to_h = @hash_table
end

# использование: доступ к вложенному конфигу через точку
config = DeepStruct.new(yaml_hash)
config.ads.menu.enabled   # вместо config['ads']['menu']['enabled']
```

## Как работает
- `DeepStruct` рекурсивно превращает вложенные хеши в объекты с доступом через точку — удобно для YAML-конфигов.
- Хранит и «структурную» (`@table`), и «сырую» (`@hash_table`) версии → `to_h` вернёт исходный хеш.
- Другие утилиты: `OrderedSet` (множество с порядком), `String` (доменные методы: `contains_russian?`, `to_underscore`), `NamedLogger` (отдельные лог-файлы под подсистемы).

## Зачем / где полезно
Мелкие сквозные хелперы, не вписывающиеся в app/-слои. `DeepStruct` — для конфигов/JSON-ответов с доступом через точку. Держи такое в `lib/`, не засоряя модели. Расширения ядра (`String`) — точечно и осознанно.
