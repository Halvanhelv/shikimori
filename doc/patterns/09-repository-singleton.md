# 09 — Repository (Singleton + Enumerable)

## Что
Справочные данные (жанры, студии, правила ачивок) грузятся **один раз** в память, дальше — лукапы O(1) без БД.

## Реальный код
```ruby
class RepositoryBase
  include Singleton          # один инстанс на класс
  include Enumerable         # map/select/find/sort_by бесплатно через #each
  attr_implement :scope      # потомок ОБЯЗАН задать scope
  delegate :[], to: :collection

  def each = collection.each_value { |e| yield e }

  def find(*args)
    id = args[0].to_i
    collection[id] || (reset && collection[id]) || raise(ActiveRecord::RecordNotFound)
  end

  def reset = (@collection = nil; true)
  def self.find(*args) = instance.find(*args)   # фасад над singleton
  private def collection = @collection ||= scope.index_by(&:id)  # {id => obj}
end

class AnimeGenresV2Repository < RepositoryBase
  private def scope = GenreV2.where(entry_type: Types::GenreV2::EntryType['Anime']).order(:position)
end
```

`NekoRepository` — то же, но источник YAML, а не БД:
```ruby
def config = @config ||= YAML.load_file(CONFIG_FILE, aliases: true)
def collection = @collection ||= config.map { |r| Neko::Rule.new(r.to_h) }.sort_by(&:sort_criteria)
```

## Как работает
- `index_by(&:id)` → хеш для лукапа за O(1).
- Паттерн «промах → `reset` → повтор»: если id не нашёлся, кеш сбрасывается (вдруг устарел), грузится заново, ещё попытка.
- `self.find` делегирует в singleton → `StudiosRepository.find(1)`.

## Зачем / где полезно
Lookup-таблицы (страны, валюты, статусы), feature-флаги, конфиги-правила. Read-heavy + write-rare.

⚠️ Singleton живёт в памяти процесса — после правки нужен `reset` или рестарт воркеров.
