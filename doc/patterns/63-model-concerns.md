# 63 — Concerns: режем толстую модель

## Простыми словами
Большая модель (Anime — сотни строк) разбита на «кусочки поведения» — concern'ы. Каждый concern отвечает за одну тему: комментарии, версии, клубы. Модель просто подключает нужные.

## Реальный код
```ruby
class Anime < ApplicationRecord
  include AniManga            # общее для аниме и манги
  include TopicsConcern       # авто-генерация форум-темы (см. 16)
  include VersionsConcern     # трекинг правок (см. 12)
  include ClubsConcern
  include FavouritesConcern
  include RknConcern          # цензура/копирайт (см. 97)
end

# concern:
module TopicsConcern
  extend ActiveSupport::Concern
  included do
    has_one :topic, as: :linked
  end
  def generate_topic = # ...
end
```

## Как работает
- `extend ActiveSupport::Concern` + блок `included do ... end` — внутри обычный код модели (связи, скоупы, колбэки).
- Модель `include`'ит несколько concern'ов → получает их связи и методы.
- Поведение сгруппировано по теме, а не свалено в один файл.

## 🆚 Классический Rails
**Обычно:** вся логика в одном файле модели на 800 строк — «god object», где перемешаны комментарии, версии, поиск, статистика. Тяжело читать и менять.
**Здесь:** каждая тема — отдельный concern в `app/models/concerns`. Модель = список подключённых способностей.

## Бенефиты
- Модель читается как оглавление: видно, что она «умеет».
- Связанное поведение — в одном месте, переиспользуется (тот же `TopicsConcern` в Anime, Character, Review).
- Меньше конфликтов при правках (разные люди — разные concern'ы).

## Когда не надо
Не превращай concern в свалку «чтобы вынести из модели» — это просто прячет god object. Concern должен быть **связной** темой. Иногда лучше отдельный объект (service/value object), а не модуль-примесь.
