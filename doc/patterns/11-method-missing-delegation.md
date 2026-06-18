# 11 — `method_missing`-делегирование

## Что
Тонкая модель хранит сухой факт в БД, а всё «как показать/что значит» делегирует через `method_missing` в объект-правило из памяти.

## Реальный код
```ruby
class Achievement < ApplicationRecord
  belongs_to :user
  delegate :cache_key, to: :neko

  def respond_to_missing?(*args) = super || neko.send(:respond_to_missing?, *args)
  def method_missing(method, *args, &block) = neko.send(method, *args, &block)

  private
  def neko = @neko ||= NekoRepository.instance.find(neko_id, level)
end
```

## Как работает
- В БД лежит только `neko_id` + `level` (факт «у юзера ачивка X уровня Y»).
- `title`, `image`, `hint`, `threshold` приходят из `Neko::Rule` (YAML-правило в памяти, см. [09](09-repository-singleton.md)).
- `method_missing` ловит любой неопределённый метод и шлёт его правилу.
- `respond_to_missing?` держит `respond_to?` честным (важно для Rails-хелперов, форм).

## Зачем / где полезно
Когда у записи мало собственных данных, но много производной логики из общего справочника/конфига. БД-схема остаётся тощей, поведение меняется правкой YAML, не миграцией.

⚠️ `method_missing` без `respond_to_missing?` ломает интроспекцию — всегда определяй оба.
