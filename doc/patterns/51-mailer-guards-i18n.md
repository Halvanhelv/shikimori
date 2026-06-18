# 51 — Mailer: guard-условия + i18n по локали получателя

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
