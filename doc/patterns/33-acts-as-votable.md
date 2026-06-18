# 33 — `acts_as_votable` (голоса/лайки)

## Что
Голосование за/против на любой модели с денормализованным кешем счётчиков.

## Реальный код
```ruby
class Critique < ApplicationRecord
  acts_as_votable cacheable_strategy: :update_columns
end

# в турнирах счётчики переиспользуются как голоса сторон матча:
# cached_votes_up = left_votes, cached_votes_down = right_votes
```

## Как работает
- Даёт `votable.liked_by(user)`, `votable.votes_for`, подсчёт за/против.
- `cacheable_strategy: :update_columns` — держит счётчики (`cached_votes_up/down`) в синхроне через прямой `update_columns` (быстро, без колбэков).
- Голоса полиморфны → одна механика для критик, обзоров, коллекций, матчей.

## Зачем / где полезно
Лайки/рейтинги/голосования. Денормализованный кеш счётчиков → не считаешь `COUNT` на каждый рендер. Применяется в турнирах ([13](13-contests-tournament.md)) для подсчёта голосов сторон.
