# 26 — Права по возрасту аккаунта (анти-спам)

## Что
Авторизация на `cancancan`, собранная модульно из ролей. Фичи открываются не сразу, а по мере «вылёживания» аккаунта — мягкий анти-спам.

## Реальный код
```ruby
class Ability
  include CanCan::Ability
  prepend Draper::CanCanCan

  def initialize(user)
    merge Abilities::User.new(user)                               # склейка ролей
    merge Abilities::ForumModerator.new(user) if user.forum_moderator?
  end
end

class Abilities::User
  def initialize(user)
    unless user.banned?
      topic_abilities   if user.week_registered?   # форум — через неделю
      comment_abilities if user.day_registered?    # комменты — через сутки
      review_abilities  if user.day_registered?
    end
  end
end

# User:
def day_registered?  = created_at + 1.day  <= Time.zone.now
def week_registered? = created_at + 1.week <= Time.zone.now
```

## Как работает
- Один `Ability`, права склеиваются из ролевых модулей через `merge`.
- Time-gating: новый аккаунт не может сразу спамить форум; права открываются постепенно.
- Иерархия: admin > moderator > user через композицию модулей.

## Зачем / где полезно
Анти-спам мягче бана: одноразовый аккаунт не успевает развернуться, живой юзер растёт в правах. Модульная авторизация для иерархии ролей. Аналог — `action_policy`/`pundit` (1 политика на ресурс).
