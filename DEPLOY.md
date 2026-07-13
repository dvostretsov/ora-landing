# Деплой лендинга ORA + настройка email

## Часть 1: Деплой на Vercel

### Шаг 1 — Загрузить файл в GitHub

1. Создайте репозиторий на github.com (например `ora-landing`)
2. Загрузите файл `index.html` в корень репозитория

### Шаг 2 — Подключить Vercel

1. Зайдите на [vercel.com](https://vercel.com) → **Add New → Project**
2. Выберите репозиторий `ora-landing`
3. Framework Preset: **Other** (просто статика)
4. Нажмите **Deploy**

Vercel выдаст временный URL вида `ora-landing.vercel.app`.

---

## Часть 2: Подключить домен ora-app.io

### Шаг 2.1 — Добавить домен в Vercel

1. В Vercel → ваш проект → **Settings → Domains**
2. Введите `ora-app.io` → **Add**
3. Vercel покажет нужные DNS-записи (обычно это A-запись или CNAME)

### Шаг 2.2 — Настроить DNS в Railway

Railway покупает домены через встроенный регистратор. Чтобы управлять DNS-записями:

1. Зайдите в [Railway Dashboard](https://railway.app) → ваш проект → **Settings → Domains**
2. Найдите домен `ora-app.io`
3. Перейдите в **DNS Management** (или **Manage DNS**)
4. Добавьте записи, которые показал Vercel:

| Тип | Хост | Значение |
|-----|------|---------|
| A   | @    | `76.76.21.21` (IP Vercel) |
| CNAME | www | `cname.vercel-dns.com` |

> ⚠️ Точные значения берите из Vercel — они могут отличаться.

Если Railway не даёт доступ к DNS напрямую, **рекомендую перевести домен на Cloudflare** (бесплатно):
1. Создайте аккаунт на [cloudflare.com](https://cloudflare.com)
2. Add Site → введите `ora-app.io`
3. Cloudflare покажет nameservers (ns1.cloudflare.com и ns2.cloudflare.com)
4. В Railway смените NS-серверы домена на cloudflare-овские
5. Дальше управляйте DNS в Cloudflare — удобнее и быстрее

---

## Часть 3: Настройка формы обратной связи (Formspree)

### Шаг 3.1 — Создать форму

1. Зайдите на [formspree.io](https://formspree.io) → New Form
2. Укажите email для получения сообщений: `hello@ora-app.io` (или ваш личный пока email не настроен)
3. Скопируйте Form ID (выглядит как `xpwzabcd`)

### Шаг 3.2 — Вставить ID в index.html

Найдите в `index.html` строку:
```html
action="https://formspree.io/f/REPLACE_ME"
```

Замените `REPLACE_ME` на ваш Form ID:
```html
action="https://formspree.io/f/xpwzabcd"
```

---

## Часть 4: Настройка email noreply@ora-app.io (SendGrid)

### Шаг 4.1 — Создать аккаунт SendGrid

1. Зайдите на [sendgrid.com](https://sendgrid.com) → Sign Up (Free tier: 100 писем/день)
2. Подтвердите email

### Шаг 4.2 — Добавить домен-отправитель

1. SendGrid → **Settings → Sender Authentication → Domain Authentication**
2. DNS host: выберите **Other** (или Cloudflare если перенесли)
3. Domain: `ora-app.io`
4. SendGrid покажет DNS-записи для добавления (2-3 CNAME)

### Шаг 4.3 — Добавить DNS-записи SendGrid

Добавьте в DNS-менеджер (Railway или Cloudflare) записи от SendGrid. Примерно так:

| Тип  | Хост                          | Значение                          |
|------|-------------------------------|-----------------------------------|
| CNAME | `em1234.ora-app.io`          | `u1234567.wl.sendgrid.net`        |
| CNAME | `s1._domainkey.ora-app.io`   | `s1.domainkey.u1234567.wl.sendgrid.net` |
| CNAME | `s2._domainkey.ora-app.io`   | `s2.domainkey.u1234567.wl.sendgrid.net` |

> Точные записи берите из SendGrid — они уникальны для вашего аккаунта.

### Шаг 4.4 — Добавить DMARC (защита от спуфинга)

Добавьте TXT-запись:

| Тип | Хост           | Значение |
|-----|----------------|---------|
| TXT | `_dmarc.ora-app.io` | `v=DMARC1; p=quarantine; rua=mailto:dmarc@ora-app.io` |

### Шаг 4.5 — Получить API ключ SendGrid

1. SendGrid → **Settings → API Keys → Create API Key**
2. Permissions: **Restricted Access → Mail Send → Full Access**
3. Скопируйте ключ — он покажется только один раз

### Шаг 4.6 — Добавить в .env backend

```bash
SMTP_HOST="smtp.sendgrid.net"
SMTP_PORT="587"
SMTP_USER="apikey"
SMTP_PASSWORD="SG.xxxxxxxxxxxxxxxx"   # ключ из шага 4.5
EMAIL_FROM="noreply@ora-app.io"
```

Это значения уже предусмотрены в `.env.example` проекта.

### Шаг 4.7 — Верифицировать домен

В SendGrid → Sender Authentication → нажмите **Verify** рядом с ora-app.io.
Если DNS-записи правильно добавлены — все три CNAME станут зелёными ✅.

После верификации письма с `noreply@ora-app.io` будут:
- Проходить SPF и DKIM проверки
- Не попадать в спам
- Отображаться как отправленные с вашего домена

---

## Итоговая последовательность

1. ☐ Загрузить `index.html` в GitHub
2. ☐ Создать проект на Vercel, задеплоить
3. ☐ Добавить домен ora-app.io в Vercel
4. ☐ Настроить DNS в Railway / Cloudflare (A + CNAME записи Vercel)
5. ☐ Создать форму на Formspree, вставить ID в index.html
6. ☐ Создать аккаунт SendGrid, добавить домен ora-app.io
7. ☐ Добавить DNS-записи SendGrid (CNAME x3 + DMARC TXT)
8. ☐ Верифицировать домен в SendGrid
9. ☐ Добавить SMTP-данные SendGrid в .env backend

---

## Обслуживание: ссылка на Android APK (обновлять при КАЖДОЙ новой Android-сборке)

С 2026-07-13 рядом с заглушками App Store/Google Play на лендинге есть рабочая кнопка «Бета для Android (APK)» + QR-код — обе ведут на конкретный EAS-билд, а не на «последнюю сборку» динамически. **Ссылка не обновляется сама** — при каждом новом `eas build --profile preview --platform android` (или `production`) нужно вручную обновить `index.html`.

**Единственное место для правки** — `index.html`, `<script id="android-apk-config">`, константа:
```js
const ANDROID_APK_URL = "https://expo.dev/accounts/dvostretsov/projects/ora-mobile/builds/<build-id>";
```
Взять новую ссылку из вывода `eas build` (строка `Open this link on your Android devices... to install the app:`) или со страницы https://expo.dev/accounts/dvostretsov/projects/ora-mobile/builds — самая свежая сборка сверху. Кнопка и QR-код (генерируется динамически через api.qrserver.com из этой же константы) подтянут новое значение автоматически — больше ничего в файле искать не нужно.

После правки — закоммитить и запушить `index.html` в `dvostretsov/ora-landing` (main, без pre-push hook), Vercel задеплоит автоматически.
