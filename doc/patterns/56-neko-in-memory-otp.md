# 56 — In-memory состояние neko (OTP: Agent/GenServer/Registry/poolboy)

## Простыми словами
Вместо одного большого хранилища (Redis/БД) neko держит данные в рое крошечных процессов-«человечков». Одни (правила, аниме) живут всегда и общие для всех. Другие (данные конкретного юзера) рождаются по требованию, когда юзер активен, и сами «умирают», освобождая память, когда он ушёл. Это фирменный способ Elixir/BEAM: много мелких процессов, каждый со своим кусочком состояния и авто-уборкой.

## Что
Neko держит состояние не в БД/Redis, а в рое лёгких Elixir-процессов. Два класса памяти: глобальная общая (правила+аниме) и по-юзерная эфемерная (оценки+ачивки) с авто-выгрузкой по простою.

## Дерево супервизоров
```elixir
# application.ex — порядок важен из-за :rest_for_one
children = [
  shikimori_pool,                  # hackney: пул HTTP обратно в shikimori
  Neko.Anime.Store,                # Agent: все аниме
  Neko.Rule.CountRule.Store,       # Agent: count-правила
  Neko.Rule.DurationRule.Store,    # Agent: duration-правила
  rule_worker_pool,                # poolboy: 30 воркеров расчёта
  UserRate.Store.Registry, Achievement.Store.Registry, user_handler_registry,
  UserRate.Store.DynamicSupervisor, Achievement.Store.DynamicSupervisor,
  UserHandler.DynamicSupervisor,
  cowboy
]
strategy: :rest_for_one   # упал ребёнок → перезапуск его и всех ПОСЛЕ него
```

## Кирпичи OTP
| Примитив | Роль здесь |
|--|--|
| **Agent** | держатель состояния (get/update) — все Store |
| **GenServer** | процесс с логикой — Rule.Worker, UserHandler |
| **Registry** | найти процесс по `user_id`; умер → ключ убран |
| **DynamicSupervisor** | спавн детей по требованию (store на юзера) |
| **poolboy** | фиксированный пул из 30 воркеров расчёта |

## 1. Глобальная память (раз при старте)
```elixir
# Anime.Store: грузит из API + предрасчёт длительностей
def start_link(_), do: Agent.start_link(fn -> animes() |> calc() end, name: @name)

# CountRule.Store: правила из YAML + тяжёлый предрасчёт НА ЗАГРУЗКЕ
defp calc(rules) do
  rules
  |> Calculations.calc_anime_ids(Neko.Anime.all())     # какие аниме под правило
  |> Calculations.calc_thresholds(&CountRule.threshold/1)
end

# Rule.Worker: каждый из 30 КОПИРУЕТ правила+аниме в своё состояние
def init(_), do: {:ok, {rules(), animes_by_id()}}      # параллельный расчёт без общей блокировки
```

## 2. По-юзерная память (по требованию, выгружается)
```elixir
defmodule Neko.UserRate.Store do
  use Agent, restart: :temporary        # умер → НЕ воскрешать (эфемерно)
  def start_link(_), do: Agent.start_link(fn -> %{} end)
  defp user_rates(uid), do: Neko.Shikimori.Client.get_user_rates!(uid) |> reduce_to_map
  def stop(pid), do: Agent.stop(pid)    # освобождение памяти
end

defmodule Neko.UserHandler do
  use GenServer, restart: :temporary
  def process(%{user_id: uid} = req), do: GenServer.call(via_tuple(uid), {:process, req}, @call_timeout)
  defp via_tuple(uid), do: {:via, Registry, {@registry_name, uid}}   # процесс по user_id

  def handle_info(:timeout, state) do   # простой → выгрузка памяти юзера
    Neko.UserRate.stop(state); Neko.Achievement.stop(state)
    {:stop, :normal, state}
  end
end
```

## Жизненный цикл запроса
```
POST /user_rate {user_id}
  → UserHandler по user_id (Registry): нет → DynamicSupervisor спавнит
  → UserRate/Achievement Store этого юзера (спавн по требованию)
      → пусто: GET shikimori API → списки в Agent (кеш в памяти)
  → checkout Rule.Worker из poolboy (1 из 30)
  → пересечение MapSet'ов (оценки ∩ аниме правила) → дельта
  → ответ {added, updated, removed}; @recv_timeout сбрасывается новым запросом
  → простой N → UserHandler :timeout → stop сторов → память свободна
```

## Зачем / где полезно
- **Общее read-heavy** (правила/аниме): грузи раз, держи в Agent, копируй в воркеры для параллельного CPU.
- **На всех не влезает** (данные юзеров): процесс-на-сущность (Registry + DynamicSupervisor), ленивый спавн, авто-eviction по простою.
- **poolboy** ограничивает CPU-параллелизм независимо от числа юзеров (backpressure).
- **`restart: :temporary`** на эфемерных процессах: не воскрешаем, пересоздадутся при следующем запросе; глобальные сторы переживают краши.

Это «BEAM-способ» кеша: рой маленьких процессов со своим куском состояния и авто-очисткой по неактивности вместо одного большого Redis. См. [54 — двунаправленность](54-neko-bidirectional.md), [17 — ачивки](17-achievements-neko.md).

## 🆚 Классический Rails
**Обычно (наивно):** состояние держат в одном общем хранилище — Redis/Memcached/таблица БД. Кеш на юзера живёт «глобально», чистится по TTL вслепую, а параллельный расчёт упирается в GIL Ruby (потоки не дают настоящего CPU-параллелизма).
**Здесь:** состояние — в изолированных процессах BEAM. Общее (правила/аниме) копируется в пул воркеров для **настоящего** параллельного расчёта; данные юзера живут в его личном процессе и выгружаются по простою точечно, а не по слепому TTL.

## Бенефиты
- Настоящий параллелизм расчёта (BEAM-планировщик), а не упор в GIL.
- Память освобождается точно по неактивности юзера (процесс сам себя гасит), а не по угадыванию TTL.
- Падение процесса одного юзера не роняет остальных (изоляция + супервизоры).
- Бэкпрешер из коробки (пул из 30 воркеров) — нагрузка ограничена независимо от числа юзеров.
