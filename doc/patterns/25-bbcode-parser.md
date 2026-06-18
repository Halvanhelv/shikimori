# 25 — BBCode-парсер

## Что
Свой движок разметки: 50+ тегов (`[spoiler]`, `[anime=123]`, `[poster]`, цитаты, опросы) с двупроходной обработкой и умными ссылками на сущности.

## Реальный код
```ruby
# двупроходность: pre-process → основной разбор → post-process
def bb_codes(text)
  tags = PRE_POST_PROCESS_TAG.map(&:new)
  text = tags.inject(text) { |memo, tag| tag.preprocess(memo) }   # код, смайлы
  text = parse(text)                                              # основной проход
  tags.reverse.inject(text) { |memo, tag| tag.postprocess(memo) }
end

# умная ссылка на сущность: [Cowboy Bebop] → находит в БД, линкует
def match_entry(name)
  if name.contains_russian? then find_russian(name, reversed_name)   # по полю :russian
  else find_name(name, reversed_name)                                # по полю :name (англ)
  end
end

# санитайзинг забаненных доменов
text.gsub(BANNED_DOMAINS, '[deleted]')
```

## Как работает
- **Два прохода**: pre (защищаем код/смайлы от разбора) → parse → post (финальная сборка). Порядок post — обратный pre.
- **Контекстный поиск в БД**: распознаёт язык названия и ищет по нужному полю.
- **Безопасность**: вырезает спам-домены, экранирует пользовательский ввод.
- Ссылки несут метаданные для тултипов (`data-attrs` с JSON).

## Зачем / где полезно
Любая пользовательская разметка с расширяемым набором тегов: форумы, комменты, wiki. Учит: расширяемый парсер + двупроходность + контекстные запросы + санитайзинг.
