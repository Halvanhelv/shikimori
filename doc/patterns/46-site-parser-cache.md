# 46 — Парсер-скрапер с файловым кешем

## Что
Базовый класс для скраперов внешних сайтов (MAL, торрент-трекеры): загрузка через прокси + персистентный кеш на диске + потокобезопасное сохранение.

## Реальный код
```ruby
class SiteParserWithCache
  def initialize
    @mutex = Mutex.new
    @cache_name = self.class.name.to_underscore
    @cache_path = "#{ENV['HOME']}/.#{@cache_name}.yml"
    load_cache
  end

  def save_cache
    @mutex.synchronize do                  # потокобезопасно
      data = YAML.dump(@cache)
      File.write(@cache_tmp_path, data)    # сначала во временный
      `cp #{@cache_tmp_path} #{@cache_path}` # потом атомарно копируем
    end
  end

  def self.fix_name(name)
    name&.downcase&.gsub(/[-:,.~"]/, '')&.gsub(/  +|　/, ' ')&.strip   # нормализация для матчинга
  end

  private

  def get(url, required_text = @required_text)
    Proxy.get(url, timeout: 30, required_text:, no_proxy: Rails.env.test?)  # через прокси-пул
  end
end
```

## Как работает
- Кеш в YAML на диске — переживает рестарты, не дёргает внешний сайт повторно за тем же.
- Запись через temp-файл + копирование → меньше шанс повредить кеш при падении.
- `Mutex` — кеш пишется из нескольких потоков безопасно.
- `Proxy.get` — загрузка через пул прокси (обход блокировок/лимитов), `required_text` проверяет что страница валидна.
- `fix_name` нормализует названия для фаззи-матчинга с записями в БД.

## Зачем / где полезно
Скрапинг/импорт из внешних источников: дорого и нестабильно. Кеш + прокси + нормализация + потокобезопасность — обязательная обвязка. Запускается из фоновых воркеров с лимитом ([20](20-sidekiq-limit-fetch.md)) и локом ([21](21-redis-mutex.md)).
