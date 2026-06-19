# Каталог полезных паттернов Shikimori

Разбор нетипичных и поучительных приёмов из кодовой базы shikimori. Один приём = один файл. Каждый файл: что это, реальный код, как работает, где пригодится вне проекта.

## Ruby / архитектура
- [01 — method_object (callable-объекты)](01-method-object.md)
- [02 — pattr/vattr/rattr_initialize](02-attr-extras-initializers.md)
- [03 — instance_cache через prepend](03-instance-cache-prepend.md)
- [04 — DslAttribute (конфиг класса)](04-dsl-attribute.md)
- [05 — dry-types на границах](05-dry-types-boundaries.md)
- [06 — ShallowAttributes value-объекты](06-shallow-attributes.md)
- [07 — QueryObjectBase (чейн над relation)](07-query-object-base.md)
- [08 — Null Object](08-null-object.md)
- [09 — Repository (Singleton + Enumerable)](09-repository-singleton.md)
- [10 — Value Objects](10-value-objects.md)
- [11 — method_missing-делегирование](11-method-missing-delegation.md)

## Доменные системы
- [12 — Версионирование контента (wiki-модерация)](12-versions-moderation.md)
- [13 — Турниры (Strategy + автомат)](13-contests-tournament.md)
- [14 — Рекомендации (коллаб. фильтрация)](14-recommendations.md)
- [15 — DSL фильтров каталога](15-animes-query-dsl.md)
- [16 — Авто-генерация топиков (полиморфизм + STI)](16-topic-generation.md)
- [17 — Ачивки через внешний Elixir-сервис](17-achievements-neko.md)

## Инфраструктура / cross-cutting
- [18 — Chewy: ES с языковыми анализаторами](18-chewy-elasticsearch.md)
- [19 — sidekiq-unique-jobs (дедуп задач)](19-sidekiq-unique-jobs.md)
- [20 — sidekiq-limit_fetch (лимит очередей)](20-sidekiq-limit-fetch.md)
- [21 — redis-mutex (распределённые локи)](21-redis-mutex.md)
- [22 — aasm (конечные автоматы)](22-aasm-state-machines.md)
- [23 — Camo: прокси картинок](23-camo-image-proxy.md)
- [24 — Faye: realtime через websockets](24-faye-realtime.md)
- [25 — BBCode-парсер](25-bbcode-parser.md)
- [26 — Права по возрасту аккаунта](26-authorization-age-gating.md)
- [27 — Migrators (рефакторинг данных)](27-migrators.md)
- [28 — rack-attack (адаптивный throttle)](28-rack-attack.md)
- [29 — webm-превью через ffmpeg](29-webm-ffmpeg.md)
- [30 — gon (Rails → JS)](30-gon.md)
- [31 — enumerize (типизированные enum)](31-enumerize.md)
- [32 — acts_as_list (позиционирование)](32-acts-as-list.md)
- [33 — acts_as_votable (голоса)](33-acts-as-votable.md)
- [34 — Shrine (деривативы картинок)](34-shrine-derivatives.md)
- [35 — xxhash для cache-key](35-xxhash-cache-keys.md)
- [36 — ar_lazy_preload (авто-фикс N+1)](36-ar-lazy-preload.md)
- [37 — clockwork (планировщик в процессе)](37-clockwork.md)
- [38 — Русский i18n + словоформы](38-i18n-russian.md)
- [39 — Кеширование на границе декоратора](39-decorator-caching.md)

## API / контроллеры / представление
- [40 — GraphQL: лимиты глубины/сложности](40-graphql-query-limits.md)
- [41 — Сериализаторы (слой JSON-вывода)](41-serializers.md)
- [42 — Кастомные валидаторы (EachValidator)](42-custom-validators.md)
- [43 — ViewObject (vs Decorator)](43-view-objects.md)
- [44 — Доменные ошибки (иерархия исключений)](44-domain-errors.md)
- [45 — Композиция контроллера из concern'ов](45-controller-concerns.md)
- [46 — Парсер-скрапер с файловым кешем](46-site-parser-cache.md)
- [47 — Утилиты в lib/ (DeepStruct и др.)](47-lib-extensions.md)

## Полнота: остальные слои
- [48 — GraphQL-резолвер: lookahead + lazy_preload](48-graphql-resolver-lookahead.md)
- [49 — Политики через static_facade](49-policy-static-facade.md)
- [50 — Автокомплит: тонкий контроллер + Search-объект](50-autocomplete-search-concern.md)
- [51 — Mailer: guard'ы + i18n по локали получателя](51-mailer-guards-i18n.md)
- [52 — Хелперы: время + русская плюрализация](52-helpers-russian-pluralize.md)

## Архитектура стека (как взаимодействуют проекты)
- [53 — Архитектура стека (4 сервиса)](53-stack-architecture.md)
- [54 — Двунаправленная интеграция с neko](54-neko-bidirectional.md)
- [55 — Аутентификация между сервисами (токены + HMAC)](55-service-tokens.md)
- [56 — In-memory состояние neko (OTP)](56-neko-in-memory-otp.md)
- [57 — Set-based diff (added/removed/updated)](57-set-based-diff.md)
- [58 — Параллельная загрузка + pattern-matching диспетч](58-parallel-load-pattern-match.md)
- [59 — Внутренности camo-сервера (безопасный прокси)](59-camo-server-internals.md)

## Модели и данные (для новичка: зачем + сравнение с обычным Rails)
- [60 — Декораторы вместо хелперов](60-decorators-vs-helpers.md)
- [61 — STI (наследование на одной таблице)](61-sti.md)
- [62 — Полиморфные связи](62-polymorphic-associations.md)
- [63 — Concerns: режем толстую модель](63-model-concerns.md)
- [64 — Хелперы в ApplicationRecord](64-application-record-helpers.md)
- [66 — Сортировка по своему порядку (Arel + CASE)](66-arel-case-ordering.md)
- [67 — Массивы PostgreSQL в колонке](67-postgres-array-columns.md)
- [68 — jsonb + value-объект](68-jsonb-value-objects.md)
- [80 — Денормализованные счётчики](80-counter-cache.md)
- [96 — has_many through с богатой join-моделью](96-has-many-through-rich-join.md)
- [99 — Rails enum vs dry-types enum](99-enum-vs-dry-types.md)

## Бизнес-логика и фон
- [65 — Search-объекты (Elasticsearch)](65-search-objects.md)
- [69 — Singleton-сервисы](69-singleton-services.md)
- [81 — Операции, вызывающие операции](81-operations-composition.md)
- [82 — BanDuration: парсинг и формат значения](82-ban-duration-vo.md)
- [83 — Пул прокси для скрапинга](83-proxy-pool-scraping.md)
- [88 — AASM-колбэки переходов](88-aasm-callbacks.md)
- [89 — Цензура/RKN-фильтрация](89-censoring-rkn.md)
- [90 — Подводный камень мемоизации](90-memoization-caveat.md)
- [91 — Скоупы vs Query-объекты](91-scopes-vs-query-objects.md)
- [98 — Отложенный бродкаст](98-broadcast-delay.md)

## Инфраструктура и конфиг
- [70 — NamedLogger (отдельные логи)](70-named-logger.md)
- [71 — .delay (отложить вызов в фон)](71-sidekiq-delay.md)
- [72 — Повторы при сбоях (retry)](72-retriable-workers.md)
- [73 — Секреты и конфиг вне кода](73-rails-secrets.md)
- [92 — Procfile + Overmind](92-procfile-overmind.md)
- [97 — Конфиг-маппинг как константа](97-config-as-constant.md)

## API, авторизация, переводы
- [74 — Translation concern](74-translation-concern.md)
- [75 — Doorkeeper (OAuth)](75-doorkeeper-oauth.md)
- [76 — responders (respond_with)](76-responders.md)
- [77 — apipie (доки из кода)](77-apipie-docs.md)
- [78 — Jbuilder (JSON-вьюхи)](78-jbuilder-views.md)
- [79 — Человекочитаемые URL (to_param)](79-to-param-friendly-urls.md)

## Фронтенд
- [93 — Server-rendered + Turbolinks](93-turbolinks-ssr.md)
- [94 — Островки Vue](94-vue-islands.md)
- [95 — i18n-js (переводы в JS)](95-i18n-js.md)

## Кросс-языковые приёмы (Elixir / Node)
- [84 — ExConstructor (структура из хеша)](84-exconstructor-elixir.md)
- [85 — Сменный адаптер через @behaviour](85-elixir-behaviour-adapter.md)
- [86 — Plug-конвейер](86-plug-pipeline.md)
- [87 — Faye-расширения (middleware сообщений)](87-faye-extensions.md)

## 🧭 Итог
- [100 — Навигатор для новичка: когда что применять](100-newcomer-decision-guide.md)
