# 64 — Хелперы в ApplicationRecord (общая база моделей)

## Простыми словами
`ApplicationRecord` — общий «родитель» всех моделей. Туда кладут утилиты, нужные всем: безопасный разбор id, обработка огромных выборок по частям.

## Реальный код
```ruby
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true

  class << self
    # обрабатывает большую выборку батчами, не загружая всё в память
    def fetch_raw_data(sql, batch_size)
      offset = 0
      begin
        batch = connection.select_all("#{sql} LIMIT #{batch_size} OFFSET #{offset}")
        batch.each { |row| yield row }
        offset += batch_size
      end until batch.empty?
    end

    # защита от bigint-переполнения в .where(id: огромное_число)
    def fix_id(id)
      int_id = id.is_a?(String) ? Integer(id) : id
      (0..(2_147_483_647 * 10_000)).cover?(int_id) ? int_id : nil
    end
  end
end
```

## Как работает
- `fetch_raw_data` гонит сырой SQL по `LIMIT/OFFSET` батчами и `yield`'ит строки → можно обработать миллионы строк, держа в памяти только batch.
- `fix_id` отсекает заведомо невалидные/переполняющие id до запроса (Postgres падает на `WHERE id = 11111111111111111111`).

## 🆚 Классический Rails
**Обычно:** `Model.all.each` грузит всё в память (ООМ на больших таблицах); `find_each` помогает, но грузит ActiveRecord-объекты (дорого). Невалидный id → 500 от Postgres.
**Здесь:** сырые строки батчами (без оверхеда AR-объектов) + предохранитель на id в общей базе → защита для всех моделей разом.

## Бенефиты
- Утилита один раз в базе → доступна всем моделям.
- `fetch_raw_data` — память не взрывается на огромных выгрузках.
- `fix_id` — единая защита от мусорных id вместо проверок в каждом контроллере.

## Когда не надо
Для обычных выборок хватает `find_each`/`in_batches` из Rails. Сырой SQL — только когда AR-оверхед реально мешает.
