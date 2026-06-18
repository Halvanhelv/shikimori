# 34 — Shrine (деривативы картинок)

## Что
Загрузка файлов с генерацией нескольких размеров/форматов на лету (постеры в webp: main/preview/mini в 1x и 2x).

## Реальный код
```ruby
class Uploaders::PosterUploader < Shrine
  include ImageProcessing::MiniMagick

  plugin :derivatives, create_on_promote: true
  plugin :store_dimensions, analyzer: :mini_magick
  plugin :pretty_location

  Attacher.derivatives do |original|
    magick = ImageProcessing::MiniMagick.source(original).saver(quality: 94)
    {
      main_2x:    magick.resize_to_limit!(...),   # генерит набор размеров
      main:       magick.resize_to_limit!(...),
      preview_2x: ...,
      preview:    ...,
      mini:       ...
    }
    # + конвертация в webp с фолбэками
  end
end
```

## Как работает
- `plugin :derivatives` — из оригинала строит набор производных при «промоушене» файла.
- `create_on_promote: true` — деривативы генерятся когда файл финально сохраняется.
- `store_dimensions` — пишет размеры в метаданные.
- MiniMagick + webp + качество 94 → лёгкие быстрые картинки.

## Зачем / где полезно
Аватары, постеры, превью — где нужно несколько размеров/форматов. Современная альтернатива Active Storage с тонким контролем pipeline'а обработки.
