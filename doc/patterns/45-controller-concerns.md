# 45 — Композиция контроллера из concern'ов

## Простыми словами
В базовом контроллере со временем копится много «сквозной» логики: локаль, цензура, хлебные крошки, пагинация, обработка ошибок. Вместо одного раздутого файла её режут на маленькие модули (concern'ы) — каждый про один аспект — и подключают по списку. Это как собрать контроллер из кубиков LEGO, а не лепить монолит; причём порядок кубиков иногда важен.

## Что
`ApplicationController` собран из множества узких concern'ов, каждый отвечает за один cross-cutting аспект. Порядок подключения важен.

## Реальный код
```ruby
class ApplicationController < ActionController::Base
  include ErrorsConcern        # must be the first
  include CensoredConcern      # must be the second
  include LocaleConcern        # must be the third
  include AgeRestrictionsConcern
  include BreadcrumbsConcern
  include DomainsConcern
  include OpenGraphConcern
  include PaginationConcern
  include StorableLocationConcern
  # ...

  before_action :touch_last_online
  before_action :force_no_cache, unless: :user_signed_in?
  before_action :force_ssl_unless_cf if Rails.env.production?

  def current_user
    @decorated_current_user ||= super.try(:decorate)   # юзер сразу декорирован
  end
end
```

## Как работает
- Каждый аспект (ошибки, локаль, цензура, хлебные крошки, пагинация, og-теги) — отдельный модуль.
- Порядок include значим: `ErrorsConcern` первый (ловит всё), `LocaleConcern` третий (язык до рендера).
- `current_user` переопределён → возвращает **декорированного** юзера везде.
- Тонкие фичи в `before_action`: онлайн-статус, заголовки кеша по user-agent, форс SSL мимо Cloudflare.

## Зачем / где полезно
Когда в базовом контроллере копится много сквозной логики — режь на concern'ы по одному аспекту. Читаемо, тестируемо по отдельности, переиспользуемо. Документируй порядок там, где он критичен.

## 🆚 Классический Rails
**Обычно:** сквозную логику (`before_action`'ы, обработка ошибок, локаль) пишут прямо в `ApplicationController` — он быстро раздувается в «бога» на сотни строк, который тяжело читать и тестировать.
**Здесь:** `ApplicationController` собран из множества узких concern'ов, каждый про один аспект (`ErrorsConcern`, `LocaleConcern`, `CensoredConcern`...). Порядок подключения значим и задокументирован комментариями (`must be the first`).

## Бенефиты
- Базовый контроллер читается как оглавление, а не как простыня кода.
- Каждый аспект тестируется и переиспользуется по отдельности.
- Понятно, где что лежит — изменения локальны.
- Критичный порядок подключения явно зафиксирован в комментариях.
