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
