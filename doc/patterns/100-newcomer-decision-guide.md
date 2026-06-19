# 100 — Навигатор для новичка: когда что применять

Памятка-итог: пришла задача — куда смотреть. Не про «как написано в shikimori», а «как принять решение».

## Куда положить логику?
```
Логика отображения (как показать)        → Decorator (60) или ViewObject (43)
Один сценарий действия (создать/посчитать) → Operation/callable (01), композиция (81)
Сложная выборка/фильтр                    → Query-объект (07, 15); простая → scope (91)
Доступ к справочнику (жанры, правила)      → Repository (09) или Singleton (69)
Значение с правилами (деньги, даты, веса)  → Value Object (10, 06)
Проверка прав                             → Policy (49) + Ability по ролям (26)
Форма JSON для API                        → Serializer (41) или jbuilder (78)
Сложная валидация поля                    → Custom Validator (42)
```

## Как не раздуть модель?
- Тема поведения (комментарии, версии) → **concern** ([63](63-model-concerns.md)).
- Похожие типы в одной таблице → **STI** ([61](61-sti.md)).
- Связь с кем угодно → **полиморфизм** ([62](62-polymorphic-associations.md)).
- Связь со своими данными → **join-модель + through** ([96](96-has-many-through-rich-join.md)).
- Статусы/жизненный цикл → **aasm** ([22](22-aasm-state-machines.md), [88](88-aasm-callbacks.md)), не булевы флаги.

## Производительность / БД
- N+1 → `lazy_preload` ([36](36-ar-lazy-preload.md)) / `includes`.
- Частый `COUNT` → счётчик в колонке ([80](80-counter-cache.md)).
- Дорогой повторный вызов → `instance_cache` ([03](03-instance-cache-prepend.md)) (осторожно с relation — [90](90-memoization-caveat.md)).
- Список id без метаданных → массив-колонка ([67](67-postgres-array-columns.md)).
- Огромная выгрузка → батчи ([64](64-application-record-helpers.md)).
- Поиск с опечатками/языками → Chewy/ES ([18](18-chewy-elasticsearch.md), [65](65-search-objects.md)).

## Фон и конкурентность
- Тяжёлое/внешнее → фоновый Job; разовое → `.delay` ([71](71-sidekiq-delay.md)).
- Не делать дважды → unique-jobs ([19](19-sidekiq-unique-jobs.md)).
- Лимит на очередь → limit_fetch ([20](20-sidekiq-limit-fetch.md)).
- Гонка между процессами → redis-mutex ([21](21-redis-mutex.md)).
- Сбои сети → осознанный retry ([72](72-retriable-workers.md)).
- Уведомить о «дозревшем» → отложенный бродкаст ([98](98-broadcast-delay.md)).

## Границы и внешний мир
- Внешний ввод → сразу в типизированный объект (dry-types [05](05-dry-types-boundaries.md), enum vs dry-types [99](99-enum-vs-dry-types.md)).
- Внешний сервис → контракт + сменный адаптер ([85](85-elixir-behaviour-adapter.md)), секреты вне кода ([73](73-rails-secrets.md)).
- Сквозные шаги запроса → concern'ы контроллера ([45](45-controller-concerns.md)) / Plug ([86](86-plug-pipeline.md)).
- Проксирование чужого контента → паранойя на каждом слое ([23](23-camo-image-proxy.md), [59](59-camo-server-internals.md)).
- Межсервисная связь → токен/HMAC под угрозу ([55](55-service-tokens.md)).

## Главные «вкусы» проекта (отличия от ваниль-Rails)
1. **Тонкие контроллеры/модели** — логика в callable-объектах ([01](01-method-object.md)).
2. **Слои**: operation → service → query → model → policy → decorator/serializer.
3. **Типы на границах** (dry-types) вместо «строка отовсюду».
4. **Выноси неудобное для Ruby** в спец-сервисы ([53](53-stack-architecture.md)): расчёт → Elixir, сокеты → Node.
5. **Кешируй на границе вьюхи** ([39](39-decorator-caching.md)), материализуй relation ([90](90-memoization-caveat.md)).

## Принцип принятия решения
Спроси: *«это переиспользуется? тестируется без HTTP/БД? не раздувает модель/контроллер?»* Если да — выноси в подходящий слой выше. Если нет (простой одноразовый случай) — оставь в модели/контроллере, не плоди классы. **Паттерн — инструмент, не самоцель.**
