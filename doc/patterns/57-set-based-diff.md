# 57 — Set-based diff (added/removed/updated)

## Что
Вычисление дельты между старым и новым набором (какие ачивки добавить/убрать/обновить) через операции над хешами и `MapSet`, по составному ключу.

## Реальный код
```elixir
# neko: Achievement.Diff
def call(old, new) do
  ko = keyed(old); kn = keyed(new)
  %{
    added:   Map.drop(kn, Map.keys(ko)) |> Map.values() |> MapSet.new(),  # есть в new, нет в old
    removed: Map.drop(ko, Map.keys(kn)) |> Map.values() |> MapSet.new(),  # есть в old, нет в new
    updated: updated(ko, kn)
  }
end

defp keyed(achievements) do
  Enum.reduce(achievements, %{}, fn a, acc -> Map.put(acc, comparison_key(a), a) end)
end
defp comparison_key(a), do: {a.neko_id, a.level}   # составной ключ идентичности

defp updated(ko, kn) do
  old_common = Map.take(ko, Map.keys(kn)) |> Map.values() |> MapSet.new()
  new_common = Map.take(kn, Map.keys(ko)) |> Map.values() |> MapSet.new()
  MapSet.difference(new_common, old_common)   # общий ключ, но значение изменилось
end
```

## Как работает
- Оба набора превращаются в хеш `{ключ => элемент}` по **составному ключу идентичности** (`{neko_id, level}`).
- `added` = ключи только в new; `removed` = ключи только в old (через `Map.drop`).
- `updated` = ключи в обоих, но значение различается (`MapSet.difference` на пересечении ключей).
- Результат — три набора, ровно то, что нужно записать/удалить/обновить ([17](17-achievements-neko.md)).

## Зачем / где полезно
Синхронизация состояний: «было → стало», нужно применить только разницу. Везде, где пересчитываешь полный набор и хочешь минимальные изменения в БД/UI/индексе: ачивки, теги, права, подписки. Ключ идентичности отделяет «тот же объект» от «изменившегося значения».
