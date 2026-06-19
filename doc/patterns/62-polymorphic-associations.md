# 62 — Полиморфные связи

## Простыми словами
Одна модель может «принадлежать» разным типам через одну связь. Комментарий может висеть на аниме, клубе, обзоре — но связь одна (`commentable`), а не три отдельные.

## Реальный код
```ruby
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true   # может быть Topic / User / Review / ...
end

class Topic < ApplicationRecord
  belongs_to :linked, polymorphic: true         # тема привязана к аниме/клубу/обзору
end

# в таблице две колонки: commentable_id + commentable_type
comment.commentable        # => #<Anime> или #<Club> ...
```

## Как работает
- В таблице две колонки: `*_id` (id записи) + `*_type` (имя класса).
- Rails по `commentable_type` понимает, к какой модели идти, по `commentable_id` — какую запись.
- Один и тот же `Comment` цепляется к чему угодно.

## 🆚 Классический Rails
**Обычно (без полиморфизма):** для «комментариев к аниме И к клубам» завели бы `anime_comments` и `club_comments` или кучу nullable FK (`anime_id`, `club_id`, `review_id`...) — и `if comment.anime_id then ... elsif ...`.
**Здесь:** одна таблица `comments`, одна связь, любой новый тип «комментируемого» подключается без миграции схемы.

## Бенефиты
- Один механизм комментариев/лайков/тегов для всех сущностей.
- Новый тип → просто `has_many :comments, as: :commentable`, без новых таблиц/колонок.
- Меньше дублирования логики.

## Когда не надо
Если нужна строгая ссылочная целостность на уровне БД (FK) — полиморфные FK её не дают. Тогда отдельные связи или таблицы.
