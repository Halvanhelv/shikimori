# 76 — `responders` (respond_with)

## Простыми словами
Гем убирает повторяющийся код «если сохранилось — редирект/JSON, иначе — форма с ошибкой». Пишешь `respond_with @object`, а гем сам выбирает что вернуть в зависимости от формата и успеха.

## Реальный код
```ruby
class Api::V1::UserRatesController < Api::V1Controller
  def create
    @resource.save
    respond_with @resource, serializer: UserRateFullSerializer
  end
end
```

## Как работает
- `respond_with @resource` смотрит: формат запроса (HTML/JSON), сохранился ли объект, есть ли ошибки.
- Сам строит ответ: для JSON — сериализованный объект или ошибки валидации; для HTML — редирект или ре-рендер формы.
- Меньше ручного `respond_to do |format| ...`.

## 🆚 Классический Rails
**Обычно:** в каждом экшене руками:
```ruby
respond_to do |format|
  if @r.save
    format.json { render json: @r }
  else
    format.json { render json: @r.errors, status: :unprocessable_entity }
  end
end
```
**Здесь:** одна строка `respond_with @r` делает то же по соглашению.

## Бенефиты
- Меньше шаблонного кода в каждом CRUD-экшене.
- Единообразные ответы по всему API.

## Когда не надо
Нестандартная логика ответа (хитрые статусы, кастомный JSON) — пиши явно. `respond_with` хорош для типового CRUD. В новых проектах часто предпочитают явный рендер для читаемости.
