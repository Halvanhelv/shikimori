# 16 — Авто-генерация топиков (полиморфизм + STI)

## Что
Создал аниме/клуб/обзор/конкурс → форум-тема создаётся автоматически. Тема полиморфно связана с любой «комментируемой» сущностью.

## Реальный код
```ruby
module TopicsConcern
  def generate_topic(forum_id: nil)
    if self.class < DbEntry
      Topics::Generate::EntryTopic.call(model: self, user: topic_user, forum_id:)
    else
      Topics::Generate::UserTopic.call(...)
    end
  end
end

# динамическое разрешение STI-класса по имени модели:
def topic_klass = "Topics::EntryTopics::#{@model.class.name}Topic".constantize

# Topic:
belongs_to :linked, polymorphic: true
FORUM_IDS = { 'Anime' => 1, 'Manga' => 1, 'Contest' => Forum::CONTESTS_ID, ... }
```

## Как работает
- `Topic belongs_to :linked, polymorphic: true` — одна таблица связана с аниме/клубом/обзором.
- STI: тип темы = `Topics::EntryTopics::AnimeTopic` и т.п., разрешается динамически из имени модели.
- Entry-топики переопределяют `updated_at => nil`, чтобы старые аниме-темы не «тонули» в форуме от свежих комментов.
- Бродкаст темы откладывается (постер качается в транзакции).

## Зачем / где полезно
Когда у разнородных сущностей должна быть общая «обвязка» (комментарии, лента, подписки). Полиморфизм + STI + динамическое разрешение класса. Учит, как избежать дублирования таблиц/логики под каждый тип.
