# 29 — webm-превью через ffmpeg

## Простыми словами
Чтобы показать картинку-превью для видео, из него вырезают один кадр внешней программой ffmpeg. Делается это в фоне (не заставляя пользователя ждать), а статус «в обработке / готово / ошибка» хранится у видео. Это как сдать плёнку в фотолабораторию: отдал и занимаешься своими делами, а потом забираешь готовый снимок.

## Что
Генерация превью для видео в фоновом воркере через ffmpeg, со статусом обработки через автомат.

## Реальный код
```ruby
class WebmThumbnail
  include Sidekiq::Worker
  sidekiq_options queue: :webm_thumbnails   # лимит 1 (см. 20)

  def perform(webm_video_id)
    grab_thumbnail!(download_url(webm_video), thumbnail_path)
  end

  def grab_thumbnail!(url, path)
    # Shellwords.shellescape — защита от инъекции в шелл
    `ffmpeg -y -i #{Shellwords.shellescape(url)} -ss 00:00:05 -vframes 1 #{Shellwords.shellescape(path)}`
  end
end

class WebmVideo < ApplicationRecord
  include AASM
  aasm column: 'state' do
    state :pending, initial: true
    state :processed; state :failed
    event(:process) { transitions from: [:pending, :failed], to: :processed }
  end
end
```

## Как работает
- Тяжёлый ffmpeg-вызов — в фоне, в очереди с лимитом 1 ([20](20-sidekiq-limit-fetch.md)).
- Состояние обработки — `aasm` ([22](22-aasm-state-machines.md)): pending→processed/failed.
- `Shellwords.shellescape` обязателен при шелл-вызове с пользовательскими данными.

## Зачем / где полезно
Обработка медиа (превью, транскод, OCR) через внешние бинарники. Учит: тяжёлое — в фон с лимитом, статус — автоматом, шелл — экранировать.

## 🆚 Классический Rails
**Обычно:** генерацию превью пытаются делать синхронно в запросе (пользователь висит и ждёт), а статус «готово/нет» нигде явно не хранят — приходится гадать по наличию файла.
**Здесь:** тяжёлый ffmpeg-вызов уходит в Sidekiq-воркер с лимитом параллелизма, а состояние обработки — явный автомат `aasm` (pending → processed/failed).

## Бенефиты
- Пользователь не ждёт обработки видео — она идёт в фоне.
- Очередь с лимитом 1 не даёт ffmpeg положить сервер кучей параллельных процессов.
- Явный статус обработки — видно, что готово, а что упало (и можно перезапустить).
- `Shellwords.shellescape` защищает от инъекции в шелл при пользовательских данных.
