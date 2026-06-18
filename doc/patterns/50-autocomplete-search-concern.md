# 50 — Автокомплит: тонкий контроллер + Search-объект

## Что
Контроллеры автодополнения предельно тонкие: вся логика — в отдельных `Search::*`-объектах, общий ввод — через concern.

## Реальный код
```ruby
class Autocomplete::FansubbersController < ShikimoriController
  include SearchPhraseConcern        # даёт search_phrase из params

  def index
    render json: Search::Fansubber.call(
      phrase: search_phrase,
      ids_limit: Autocomplete::AutocompleteBase::LIMIT,
      kind: params[:kind]
    )
  end
end
```

## Как работает
- Контроллер делает ровно одно: достаёт фразу (`SearchPhraseConcern`), зовёт callable Search-объект ([01](01-method-object.md)), рендерит JSON.
- Логика поиска (Elasticsearch/нормализация/лимиты) — в `Search::Fansubber`, не в контроллере.
- Общий ввод вынесен в concern → не дублируется по всем автокомплит-контроллерам.

## Зачем / где полезно
Эталон тонкого контроллера: input (concern) → callable (Search-объект) → output (JSON). Логика тестируется без HTTP, контроллеры однотипны и крошечные. Подходит для любых «лёгких» JSON-эндпоинтов (автокомплит, виджеты, лукапы).
