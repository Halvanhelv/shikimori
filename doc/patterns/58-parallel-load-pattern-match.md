# 58 — Параллельная загрузка (Task.async) + диспетч по pattern-matching

## Что
Оркестрация обработки запроса в neko: параллельная загрузка данных через `Task.async/await` и выбор действия через сопоставление с образцом, а не `if/case`.

## Реальный код
```elixir
# neko: Request.process
def process(%{user_id: user_id} = request) do
  load_user_data(request)               # параллельно тянем оценки + ачивки
  process_action(request)               # применяем действие
  new = calc_new_achievements(user_id)
  diff = calc_diff(user_id, new)        # см. 57
  save_new_achievements(user_id, new)
  diff
end

# параллельная загрузка двух источников разом:
defp load_user_data(%{user_id: uid}) do
  [Neko.UserRate, Neko.Achievement]
  |> Enum.map(&Task.async(&1, :load, [uid]))   # два запроса в shikimori API одновременно
  |> Enum.map(&Task.await(&1, @await_timeout))
end

# диспетч действия — pattern matching на структуре запроса:
defp process_action(%{action: "put", status: status} = req)
     when status in ["completed", "rewatching", "watching", "on_hold"] do
  Neko.UserRate.put(req.user_id, UserRate.from_request(req))   # активный статус → добавить
end
defp process_action(%{action: "put"} = req),    do: Neko.UserRate.delete(...)  # иначе → убрать
defp process_action(%{action: "delete"} = req), do: Neko.UserRate.delete(...)
defp process_action(%{action: "noop"}),         do: :ok
```

## Как работает
- `Task.async` запускает оба сетевых запроса параллельно, `Task.await` собирает с общим таймаутом → суммарная задержка = max(t1, t2), не t1+t2.
- `process_action` — несколько определений функции с разными образцами + guard'ы (`when status in [...]`). Рантайм выбирает подходящее. Нет ветвлений в теле.
- Бизнес-правило «put неактивного статуса = удаление из списка» выражено образцом, а не вложенным `if`.

## Зачем / где полезно
- **Параллельный fetch** независимых источников — там, где IO-bound (API, БД) и можно не ждать последовательно.
- **Pattern-matching диспетч** вместо `case/if` — каждая ветка = отдельное определение с явным условием на входе. Читаемо, расширяемо, ошибки видны сразу. В Ruby ближайший аналог — `case/in` (pattern matching 3.0+) или полиморфизм.
