# 12 — Версионирование контента (wiki-модерация)

## Что
Юзеры правят данные аниме/манги/персонажей. Правка не мутирует запись сразу, а хранится как дифф и проходит модерацию через конечный автомат. Git-подобный workflow поверх обычных моделей.

## Реальный код
```ruby
class Version < ApplicationRecord
  aasm column: 'state' do
    state :pending, initial: true
    state :accepted; state :auto_accepted; state :rejected; state :taken; state :deleted

    event :accept do
      transitions from: :pending, to: :accepted do
        after :apply_version       # применить дифф к записи
        after :assign_moderator
        success :notify_acceptance
      end
    end
  end
end
```

## Как работает
- Правка = `item_diff` (jsonb), а не прямое изменение записи.
- Полиморфные подтипы (`DescriptionVersion`, `PosterVersion`) по-разному применяют/откатывают (`apply_changes`/`rollback_changes`).
- «Значимые» правки авто-принимаются (сервис измеряет размер изменения).
- Анти-спам: лимиты на число правок, ослабляются для доверенных юзеров.

## Зачем / где полезно
Пользовательский контент с модерацией: wiki, каталоги, краудсорсинг данных. Учит: автоматы + диффы + полиморфные откаты + транзакционная безопасность. В Rails похожее делают через `audited`/`paper_trail`, но там обычно аудит, а тут полноценный propose→review→apply.
