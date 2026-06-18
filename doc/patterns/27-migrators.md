# 27 — Migrators (рефакторинг данных вне миграций)

## Что
Сложные одноразовые перекладки данных — отдельные классы в `app/migrators`, не Rails-миграции. Схема (db/migrate) и данные разделены.

## Реальный код
```ruby
class MigrateGenreV2Ids
  method_object :klass
  TEMP_ID = 876_564_321   # промежуточный id для атомарного свапа

  SPECIAL_MIGRATION_RULES = { 'Гонки' => 'Машины', 'Авангард' => 'Безумие' }

  def call
    genres_v2_repository.each do |genre_v2|
      genre_v1 = search_matching_genre_v1(genre_v2:)
      next if genre_v1.nil? || genre_v1.id == genre_v2.id
      ActiveRecord::Base.transaction { migrate(genre_v2:, to_id: genre_v1.id) }
    end
  end

  def migrate(genre_v2:, to_id:)
    migrate(genre_v2: conflicting, to_id: TEMP_ID) if conflicting   # развести конфликт
    migrate_db_entries(genre_v2:, from_id:, to_id:)
  end
end
```

## Как работает
- Логика перекладки — обычный callable-объект ([01](01-method-object.md)), запускается из консоли/задачи.
- Трюк со свапом через `TEMP_ID` решает конфликт уникальности при переназначении id (как при сложных rewrite БД).
- В транзакции — атомарно.

## Зачем / где полезно
Когда данные надо массово переразложить, а Rails-миграция стала бы громоздкой и неперезапускаемой. Отделяет «изменение структуры» от «изменения данных». Тестируется как обычный объект.
