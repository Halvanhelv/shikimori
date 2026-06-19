# Онбординг в Shikimori — для новичка

Один документ, чтобы за час понять, **как устроен проект и почему именно так**. Читается сверху вниз. Детали каждого приёма — по ссылкам в [каталог паттернов](patterns/README.md) (100 заметок).

> Если совсем нет времени — прыгай в [навигатор «когда что применять»](patterns/100-newcomer-decision-guide.md).

---

## 1. Что это за проект

Shikimori — крупный сайт-трекер аниме/манги (каталог, списки пользователей, форум, клубы, турниры, достижения). Это **не один процесс**, а четыре кооперирующих сервиса на трёх языках.

| Сервис | Язык | Зачем | Подробно |
|--|--|--|--|
| **shikimori** | Ruby on Rails | домен, API, вьюхи, очереди | весь каталог |
| **neko-achievements** | Elixir | расчёт достижений | [17](patterns/17-achievements-neko.md), [54](patterns/54-neko-bidirectional.md), [56](patterns/56-neko-in-memory-otp.md) |
| **faye-server** | Node.js | realtime через websocket | [24](patterns/24-faye-realtime.md), [87](patterns/87-faye-extensions.md) |
| **camo-server** | Node.js | прокси картинок (privacy/HMAC) | [23](patterns/23-camo-image-proxy.md), [59](patterns/59-camo-server-internals.md) |

**Главная идея стека:** монолит Rails держит домен, а «неудобное для Ruby» вынесено в спец-сервисы — тяжёлый параллельный расчёт в Elixir, тысячи сокетов и проксирование в Node. Связь — простыми HTTP/ws + общий секрет.
→ [53 — Архитектура стека](patterns/53-stack-architecture.md), [55 — токены между сервисами](patterns/55-service-tokens.md)

**Запуск:** все процессы перечислены в `Procfile`, поднимаются одной командой через Overmind.
→ [92 — Procfile + Overmind](patterns/92-procfile-overmind.md)

---

## 2. Технологии (на что смотреть)

- **Ruby 3.2 + Rails**, PostgreSQL, Redis, Memcached, **Elasticsearch** (поиск).
- **Sidekiq** — фоновые задачи. **GraphQL + REST API**. Devise + Doorkeeper (OAuth).
- Фронт: **серверный рендер (Slim) + Turbolinks + точечный Vue** + jQuery. Сборка — Shakapacker/Webpack.
  → [93 — SSR + Turbolinks](patterns/93-turbolinks-ssr.md), [94 — островки Vue](patterns/94-vue-islands.md), [95 — i18n-js](patterns/95-i18n-js.md)

---

## 3. Философия кода (самое важное для новичка)

В обычном «учебном» Rails логику пихают в **толстые контроллеры** и **толстые модели**. Здесь наоборот — **тонкие контроллеры и модели**, а логика разложена по слоям-объектам. Если запомнишь только одно — запомни это.

```
Контроллер (тонкий)
  → Operation / Service   (одно действие = один объект с .call)
      → Query-объект          (сложные выборки)
      → Repository            (справочники в памяти)
      → Model                 (данные + правила, поведение через concerns)
          → Policy                (права)
          → Decorator / ViewObject (как показать)
          → Serializer            (JSON для API)
      → Worker (Sidekiq)      (фон)
      → Chewy index           (поиск)
```

### Кирпич №1: объект-действие (callable)
Одно действие — один класс с одной кнопкой `call`. Контроллер становится одной строкой.
→ [01 — method_object](patterns/01-method-object.md), [81 — композиция операций](patterns/81-operations-composition.md)

```ruby
Comment::Create.call(params: p, faye: f)   # вся логика создания коммента внутри
```

### Кирпич №2: типы на границах
Внешний ввод (параметры, JSON) сразу превращают в строгий тип — мусор отбивается на входе, а не всплывает глубоко.
→ [05 — dry-types](patterns/05-dry-types-boundaries.md), [99 — Rails enum vs dry-types](patterns/99-enum-vs-dry-types.md)

### Кирпич №3: значение с правилами = value-объект
Деньги, даты, «уверенность», длительность бана — не голые числа, а объекты с логикой в одном месте.
→ [10 — value objects](patterns/10-value-objects.md), [06 — ShallowAttributes](patterns/06-shallow-attributes.md), [82 — BanDuration](patterns/82-ban-duration-vo.md)

### Кирпич №4: выноси неудобное
Расчёт достижений на всех юзеров — тяжёлый и параллельный → отдали Elixir. Тысячи сокетов → Node. Rails остаётся тонким посредником.
→ [53](patterns/53-stack-architecture.md), [17](patterns/17-achievements-neko.md)

---

## 4. Слой за слоем (куда что класть)

### Модели и данные
Модель — про данные и правила. Чтобы она не распухла:
- поведение режут на **concerns** по темам → [63](patterns/63-model-concerns.md)
- похожие типы в одной таблице — **STI** → [61](patterns/61-sti.md)
- связь «с кем угодно» — **полиморфизм** → [62](patterns/62-polymorphic-associations.md)
- связь со своими данными — **join-модель через `through`** → [96](patterns/96-has-many-through-rich-join.md)
- статусы/жизненный цикл — **конечный автомат (aasm)**, не булевы флаги → [22](patterns/22-aasm-state-machines.md), [88](patterns/88-aasm-callbacks.md)
- PostgreSQL-фишки: [массивы в колонке](patterns/67-postgres-array-columns.md), [jsonb + объект](patterns/68-jsonb-value-objects.md), [CASE-сортировка](patterns/66-arel-case-ordering.md)
- общие утилиты моделей — в [ApplicationRecord](patterns/64-application-record-helpers.md)

### Выборки из БД
- простой фильтр → **scope** в модели
- сложный фильтр/поиск → **query-объект** → [07](patterns/07-query-object-base.md), [15 — DSL фильтров каталога](patterns/15-animes-query-dsl.md)
- граница «scope vs query-объект» → [91](patterns/91-scopes-vs-query-objects.md)
- справочники (жанры, правила) в памяти → **Repository/Singleton** → [09](patterns/09-repository-singleton.md), [69](patterns/69-singleton-services.md)
- полнотекстовый поиск → **Chewy/Elasticsearch** → [18](patterns/18-chewy-elasticsearch.md), [65 — Search-объекты](patterns/65-search-objects.md)

### Бизнес-логика
- одно действие → [operation/callable](patterns/01-method-object.md)
- действие из шагов → [композиция операций](patterns/81-operations-composition.md)
- декларативные конструкторы → [02 — attr_extras](patterns/02-attr-extras-initializers.md)

### Представление (как показать)
- одна модель красиво → **Decorator** → [60](patterns/60-decorators-vs-helpers.md), [39 — кеш на декораторе](patterns/39-decorator-caching.md)
- блок без своей модели (меню, реклама) → **ViewObject** → [43](patterns/43-view-objects.md)
- JSON для API → **Serializer** → [41](patterns/41-serializers.md) или [Jbuilder](patterns/78-jbuilder-views.md)
- мемоизация дорогих методов → [03 — instance_cache](patterns/03-instance-cache-prepend.md) (осторожно: [90 — relation→array](patterns/90-memoization-caveat.md))

### Права и валидация
- роли и доступ → [26 — права по возрасту аккаунта](patterns/26-authorization-age-gating.md), [49 — policy-объекты](patterns/49-policy-static-facade.md)
- сложная валидация поля → [42 — кастомные валидаторы](patterns/42-custom-validators.md)
- доменные ошибки с контекстом → [44](patterns/44-domain-errors.md)

### Фон и конкурентность
- тяжёлое → Sidekiq-воркер; разовое → [.delay](patterns/71-sidekiq-delay.md)
- не делать дважды → [unique-jobs](patterns/19-sidekiq-unique-jobs.md)
- лимит на очередь → [limit_fetch](patterns/20-sidekiq-limit-fetch.md)
- гонка между процессами → [redis-mutex](patterns/21-redis-mutex.md)
- сбои сети → [осознанный retry](patterns/72-retriable-workers.md)
- расписание → [clockwork](patterns/37-clockwork.md)
- уведомить о «дозревшем» → [отложенный бродкаст](patterns/98-broadcast-delay.md)

### API и контроллеры
- тонкий контроллер из concern'ов → [45](patterns/45-controller-concerns.md)
- OAuth → [Doorkeeper](patterns/75-doorkeeper-oauth.md); ответы → [responders](patterns/76-responders.md); доки → [apipie](patterns/77-apipie-docs.md)
- GraphQL: [лимиты глубины/сложности](patterns/40-graphql-query-limits.md), [резолвер с lookahead](patterns/48-graphql-resolver-lookahead.md)
- красивые URL → [to_param](patterns/79-to-param-friendly-urls.md)

### Интеграции и инфраструктура
- realtime → [Faye](patterns/24-faye-realtime.md); картинки → [Camo](patterns/23-camo-image-proxy.md)
- секреты вне кода → [73](patterns/73-rails-secrets.md); конфиг-константы → [97](patterns/97-config-as-constant.md)
- скрапинг внешних сайтов → [парсер с кешем](patterns/46-site-parser-cache.md) + [пул прокси](patterns/83-proxy-pool-scraping.md)
- защита → [rate-limit](patterns/28-rack-attack.md), [межсервисные токены/HMAC](patterns/55-service-tokens.md)
- логи по подсистемам → [NamedLogger](patterns/70-named-logger.md)

---

## 5. Большие фичи (как изучать на примерах)

Хочешь увидеть, как кирпичи складываются в реальные системы — читай эти разборы:

1. **Достижения (neko)** — триггер → фон → внешний Elixir-мозг → запись → push. Двунаправленная интеграция, расчёт в памяти.
   → [17](patterns/17-achievements-neko.md) · [54](patterns/54-neko-bidirectional.md) · [56](patterns/56-neko-in-memory-otp.md) · [57 — set-diff](patterns/57-set-based-diff.md) · [58 — параллельная загрузка](patterns/58-parallel-load-pattern-match.md)
2. **Версионирование контента** — wiki-правки с модерацией (автомат + диффы + откаты).
   → [12](patterns/12-versions-moderation.md)
3. **Турниры** — Strategy (3 вида сетки) + автоматы + cron-оркестрация.
   → [13](patterns/13-contests-tournament.md)
4. **Рекомендации** — коллаборативная фильтрация (Pearson/SVD) руками.
   → [14](patterns/14-recommendations.md)
5. **Каталог с фильтрами** — URL → SQL через чейн фильтр-объектов.
   → [15](patterns/15-animes-query-dsl.md)
6. **Форум-темы** — авто-генерация для любой сущности (полиморфизм + STI).
   → [16](patterns/16-topic-generation.md)

---

## 6. Что отличает этот код от «ванильного» Rails

| Ваниль-Rails | Shikimori |
|--|--|
| толстый контроллер/модель | тонкие; логика в callable-объектах |
| строки/числа отовсюду | типы на границах (dry-types) |
| хелперы для вьюх | декораторы/view-объекты |
| `COUNT(*)` на лету | денормализованные счётчики |
| `ILIKE '%...%'` | Elasticsearch с анализаторами |
| всё в одном процессе | спец-сервисы (Elixir/Node) под нагрузку |
| магические числа/строки | константы-маппинги, value-объекты |

Каждая строка таблицы раскрыта в соответствующей заметке (см. [каталог](patterns/README.md)).

---

## 7. С чего начать читать код (маршрут новичка)

1. `Procfile` + [53](patterns/53-stack-architecture.md) — увидеть все сервисы.
2. `app/controllers/comments_controller.rb` → `app/operations/comment/create.rb` — как контроллер делегирует операции ([01](patterns/01-method-object.md)).
3. `app/models/anime.rb` — посмотреть `include` concern'ов ([63](patterns/63-model-concerns.md)).
4. `app/query_objects/animes/query.rb` — как строится фильтрация каталога ([15](patterns/15-animes-query-dsl.md)).
5. `app/decorators/ani_manga_decorator.rb` — как готовятся данные для вьюхи ([39](patterns/39-decorator-caching.md)).
6. [Навигатор «когда что применять»](patterns/100-newcomer-decision-guide.md) — держать под рукой при своей первой задаче.

---

## 8. Принцип принятия решений

При любой задаче спроси себя:
> *Это переиспользуется? Тестируется без HTTP/БД? Не раздувает модель/контроллер?*

- **Да** → вынеси в подходящий слой (operation / query / value-object / decorator).
- **Нет** (простой одноразовый случай) → оставь в модели/контроллере, не плоди классы.

**Паттерн — инструмент, а не самоцель.** Полный разбор «задача → паттерн» — в [навигаторе](patterns/100-newcomer-decision-guide.md).

---

📚 **Полный каталог:** [doc/patterns/README.md](patterns/README.md) — 100 заметок, каждая: простыми словами → реальный код → как работает → 🆚 классический Rails → бенефиты.
