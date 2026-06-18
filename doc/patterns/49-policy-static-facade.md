# 49 — Политики через `static_facade` (точечная авторизация)

## Что
Помимо общего `Ability` (cancancan, см. [26](26-authorization-age-gating.md)), есть мелкие политики-объекты для точечных проверок доступа, делегирующие по цепочке.

## Реальный код
```ruby
class Comment::AccessPolicy
  static_facade :allowed?, :comment, :current_user   # из attr_extras: класс-метод .allowed?

  def allowed?
    return true if own_comment?
    Commentable::AccessPolicy.allowed?(@comment.commentable, @current_user)  # делегирование выше
  end

  private

  def own_comment? = @comment.user_id == @current_user&.id
end

# вызов:
Comment::AccessPolicy.allowed?(comment, current_user)
```

## Как работает
- `static_facade :allowed?, :comment, :current_user` (attr_extras) — то же, что `method_object`, но публичный метод называется `allowed?`, а не `call`. Генерит класс-фасад `.allowed?(...)`.
- Политика отвечает на один вопрос: «можно ли?». Композируется: доступ к комменту → делегирует доступу к его `commentable` (топик/профиль/обзор).
- Цепочка политик (`Comment` → `Commentable` → конкретный тип) даёт каскадную проверку прав.

## Зачем / где полезно
Когда «можно/нельзя» сложнее ролей и зависит от связанных объектов. Маленькие composable-политики легко тестировать и переиспользовать. Сосуществует с глобальным `Ability`: cancancan — общие роли, policy-объекты — контекстные проверки.
