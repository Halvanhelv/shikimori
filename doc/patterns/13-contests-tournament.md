# 13 — Турниры (Strategy + конечный автомат)

## Что
Голосование за аниме навылет. Сетка строится по одной из трёх стратегий (double elimination / play-off / swiss), матчи проходят по состояниям, победитель проходит дальше.

## Реальный код
```ruby
class Contest::DoubleEliminationStrategy
  def total_rounds
    @total_rounds ||= Math.log(@contest.members.count, 2).ceil * 2   # двойное выбывание
  end

  def create_rounds
    total_rounds.times { |i| create_round(...) }
    @contest.rounds.each { |r| fill_round_with_matches(r) }
  end
end

# ежедневный воркер гоняет состояния:
class Contest::Progress   # created → started → freezed → finished
  # фризит матчи после finish_date, стартует pending, финишит проголосованные
end
```

## Как работает
- **Strategy**: `Contest` делегирует генерацию сетки стратегии; каждая считает раунды/пары по-своему.
- **Автоматы** на трёх уровнях: `Contest`, `ContestRound`, `ContestMatch` (aasm).
- Голоса — через `acts_as_votable` с кешем (`cached_votes_up`/`down`).
- Оркестрация — ежедневная задача через clockwork.

## Зачем / где полезно
Любой многоэтапный процесс с вариантами алгоритма: турниры, воронки, пайплайны обработки. Учит сочетанию Strategy + State Machine + cron-оркестрации. См. [22 — aasm](22-aasm-state-machines.md), [33 — acts_as_votable](33-acts-as-votable.md).
