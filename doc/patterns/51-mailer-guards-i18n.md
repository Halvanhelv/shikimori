# 51 — Mailer: guard-условия + i18n по локали получателя

## Простыми словами
Перед тем как отправить письмо, код сначала задаёт несколько вопросов: а адрес настоящий? а человек ещё не прочитал это в приложении? Если что-то не так — просто молча не шлёт. Плюс письмо пишется на языке того, кому оно адресовано (а не на языке того, кто его вызвал), а если почтовый сервер ругнулся — это не роняет всю задачу, а аккуратно записывается в журнал.

## Что
Mailer с защитными проверками (не слать сгенерированным/неактуальным адресам), письма на языке получателя, обработка SMTP-ошибок.

## Реальный код
```ruby
class ShikiMailer < ActionMailer::Base
  include Routing
  include Translation

  rescue_from Net::SMTPSyntaxError do          # битый адрес — не падаем
    user = User.find_by(email: message[:to].value)
    Messages::CreateNotification.new(user).bad_email
    NamedLogger.email.info "failed to send email to #{user.email}"
  end

  def private_message_email(message_id)
    message = Message.find_by(id: message_id)
    return unless message
    return if message.read?                    # уже прочитал в UI — не дублируем письмом
    return if generated?(message.to.email)     # технический адрес — пропускаем

    body = i18n_t('private_message_email.body',
      from_nickname: message.from.nickname,
      private_message_link: profile_dialogs_url(message.to),
      locale: message.to.locale)               # язык ПОЛУЧАТЕЛЯ
    mail(to: message.to.email, subject:, body:)
  end

  private
  def generated?(email) = !!(email.blank? || email.match?(/^generated_/))
end
```

## Как работает
- **Guard-и в начале**: `return unless/ if` отсекают ненужные отправки (прочитано, технический адрес, нет записи). Дешевле и чище, чем слать и ловить.
- **Локаль получателя**: `locale: message.to.locale` — письмо на его языке, не на языке триггера.
- **`rescue_from`**: SMTP-ошибка → пометить плохой email + лог в отдельный файл (`NamedLogger.email`), а не уронить задачу.
- `generated?` — у части юзеров служебные адреса (`generated_*`), им не пишем.

## Зачем / где полезно
Любая отправка писем: guard'ы экономят отправки и защищают от мусора, локаль получателя обязательна в мультиязычном проекте, `rescue_from` держит фон стабильным. Учит: письмо — не «просто mail()», а набор предусловий + локализация + обработка сбоев.

## 🆚 Классический Rails
**Обычно:** учебный mailer — это просто метод с `mail(to:, subject:)`. Письмо уйдёт на любой адрес, на языке по умолчанию (`I18n.locale` приложения), а SMTP-ошибка прилетит исключением и уронит фоновую задачу/реквест.
**Здесь:** в начало метода добавлены guard'ы (`return if read?`, `return if generated?(...)`), локаль берётся у получателя (`locale: message.to.locale`), а `rescue_from Net::SMTPSyntaxError` ловит битые адреса, помечает их и пишет в отдельный лог вместо падения.

## Бенефиты
- Не шлём лишних/мусорных писем (прочитанные, технические адреса) — экономия и чистота.
- Получатель видит письмо на своём языке, а не на языке того, кто триггернул отправку.
- SMTP-сбой не валит джобу: ошибка локализована, адрес помечен, есть запись в логе.
- Логика «слать или нет» читается сверху вниз короткими `return`-ами, без вложенных `if`.
