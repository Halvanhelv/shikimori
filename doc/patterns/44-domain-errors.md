# 44 — Доменные ошибки (иерархия исключений)

## Что
Структурированные доменные исключения в `app/errors`, несущие контекст (поле, значение, причину), а не голые строки. Сгруппированы по доменам (`bb_codes/`, `neko/`, `i18n/`).

## Реальный код
```ruby
class InvalidParameterError < ArgumentError
  attr_reader :field, :value

  def initialize(field, value, additional = nil)
    @field, @value, @additional = field, value, additional
  end

  def to_s = "Invalid #{@field} value \"#{@value}\"#{". #{@additional}" if @additional}"
end

# выбрасывается из фильтров каталога (см. 15):
raise InvalidParameterError.new(:season, season)
```

Примеры из каталога ошибок: `AgeRestricted`, `CopyrightedResource`, `RknBanned`, `CaptchaError`, `StateMachineRollbackError`, `MissingApiParameter`, `ForceRedirect`.

## Как работает
- Ошибка наследует подходящий базовый класс (`ArgumentError`) и **несёт данные** (`field`, `value`) — обработчик знает, что именно не так.
- Контроллер ловит конкретный тип и отдаёт нужный HTTP-статус/сообщение (через `ErrorsConcern`).
- Группировка по папкам-доменам делает набор обозримым.

## Зачем / где полезно
Вместо `raise "bad season"` — типизированное исключение с контекстом: легче ловить точечно, мапить на статусы, логировать осмысленно. `ForceRedirect`/`RknBanned` как ошибки — элегантный способ прервать поток управления из глубины и обработать наверху.
