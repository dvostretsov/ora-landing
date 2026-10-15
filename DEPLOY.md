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

## Google Play: спрос на Android вместо APK (2026-10-15)

APK-беты на лендинге больше нет (коммит `4b8e0f0`), обновлять ссылку на EAS-сборку не нужно. Кнопка «Скоро в Google Play» (`#gplay-btn`) раскрывает форму «сообщить о выходе Android» (`#android-notify`) и один раз за сессию пишет клик в Supabase `public.landing_events` (event `gplay_click`, метка кампании `ct`, язык; без IP и cookies). Код — `<script id="android-demand">`. Почты из формы приходят в Formspree с `type=notify_android`.

Считать клики: `select ct, lang, count(*) from landing_events where event='gplay_click' group by 1,2;`

**Когда приложение выйдет в Google Play:** заменить `<button id="gplay-btn">` на ссылку в стор, убрать форму `#android-notify` и скрипт `android-demand`, написать один раз всем из Formspree с `type=notify_android`. В нижней панели (`#store-bar-android`, см. часть 9) кнопку тоже заменить на ссылку.

## Метки кампаний `?s=` (2026-10-15)

Ссылки из соцсетей ведут на `www.ora-app.io/?s=<метка>` (`tg`, `ig`, `fb`, `tg-<канал>`). `<script id="campaign-links">` переносит метку во все ссылки на App Store как `ct=` вместе с provider token `pt=129070480`; установки по меткам — App Store Connect → Аналитика → Источники → Кампании. Таблица меток — в `claude/Marketing_Hub.md` (проект ORA в claude.ai).

---

## Часть 6: Языки RU/EN, формы и словесный знак (2026-10-02)

### Как устроен переключатель языков
- Русский текст — прямо в разметке (источник истины). Английский — словарь `I18N_EN` в `<script id="i18n">` внизу `index.html`.
- Элемент с текстом помечается `data-i18n="ключ"`, текст с разметкой (`<br>`, `<span>`, ссылка) — `data-i18n-html="ключ"`, атрибуты — `data-i18n-attr="placeholder:ключ"` (также `aria-label`, `alt`, `href`).
- Язык выбирается так: `?lang=en` / `?lang=ru` в адресе → ручной выбор посетителя (запоминается в браузере) → язык браузера (русский, украинский, белорусский, казахский → RU, остальные → EN).
- Ссылка на английскую версию: `https://ora-app.io/?lang=en`. Английская политика: `/privacy?lang=en#en`.

**Добавляете новый текст на страницу:**
1. В разметке — русский текст и атрибут `data-i18n="новый_ключ"`.
2. В `I18N_EN` — `"новый_ключ": "English text"`. Английский пишем для англоязычного читателя, не дословно.
3. Без шага 2 в английской версии останется русский.

**Только для одного языка:** атрибут `data-lang-only="en"` прячет элемент в русской версии (так сделаны пометка о языке приложения, «Notify me» и блок для партнёров).

### Формы (все — один Formspree `xbdvlyja`, письма на один ящик)
| Форма | Где | Тема письма | Поле `type` |
|---|---|---|---|
| Напишите нам / Get in touch | `#contact`, обе версии | стандартная Formspree | — (есть скрытое `lang`) |
| Notify me | `#notify`, только EN | ORA — notify me when English is ready | `notify_en` |
| Partners & investors | `#partners`, только EN | ORA — partner / investor inquiry | `partner` |
| Сообщить о выходе Android | `#android-notify`, обе версии (по кнопке Google Play) | ORA — сообщить о выходе Android | `notify_android` |

В формах «Notify me» и «Partners» есть скрытое поле-ловушка `_gotcha` от ботов.

**Когда выйдет английская версия приложения:** выгрузить адреса «Notify me» из Formspree и написать им один раз (на сайте обещано «One email, no newsletter»), затем убрать карточку `#notify` и строку `.tariffs__lang-note`.

### Словесный знак ORA ARTHA
- В шапке и подвале рядом с иконкой — надпись из контуров `ORA/trademark_logo/ORA_ARTHA_lockup.svg` (тот же знак, что для заявки на товарный знак). Хранится один раз как `<symbol id="ora-artha-wordmark">` в начале `<body>`, используется через `<use href="#ora-artha-wordmark"/>`.
- Цвет задаётся CSS (`fill: currentColor`), размер — классами `.nav__wordmark` (28.8 px) и `.footer__wordmark` (21.6 px). Высота подобрана по видимой части иконки; ширина = высота × 1140/649.

### Проверка перед пушем
Открыть локально `index.html?lang=en` и `?lang=ru`, проверить шапку, формы и ширину телефона (375 px). Быстрая проверка, что в английской версии не осталось русского текста, — в консоли браузера на `?lang=en`:
```js
[...document.body.querySelectorAll('*')].filter(e=>e.children.length===0&&/[А-Яа-яЁё]/.test(e.textContent)&&e.offsetParent).map(e=>e.textContent.trim())
```
Пустой массив — всё переведено.

---

## Часть 7: Логотип для подписи Gmail (2026-10-04)

- Файл `email/ora-artha-logo.png` (128×128, прозрачный фон, показывается в 64 px) — иконка для подписи Gmail Дмитрия. Подпись грузит его по адресу https://ora-app.io/email/ora-artha-logo.png.
- **Не удалять и не переименовывать:** во всех уже отправленных письмах картинка подписи пропадёт.
- Исходник подписи — `ORA/email_signature/gmail_signature.html` (репозиторий ORA-docs), там же инструкция по установке в Gmail.

---

## Часть 8: Индексация в Google (2026-10-08)
- Search Console (7 октября) сообщил «Страница с переадресацией»: ссылки вели на `/privacy.html`, а Vercel (`cleanUrls: true`) перенаправляет его на `/privacy`. Все ссылки переведены на `/privacy`.
- Добавлены `robots.txt` и `sitemap.xml` (главная + `/privacy`), а также `<link rel="canonical">` на обеих страницах.
- **Основной адрес сайта — `https://www.ora-app.io`** (Vercel перенаправляет `ora-app.io` → `www`). В canonical и sitemap — только `www`-адреса, иначе они сами указывают на переадресацию.
- **Правило:** ссылки внутри сайта — без `.html` (`/privacy`, не `/privacy.html`). Новую страницу добавлять в `sitemap.xml`.
- Переадресации `http://` → `https://` и `ora-app.io` → `www.ora-app.io` в отчёте Search Console остаются — это нормально.

---

## Часть 9: Соцсети и акцент на кнопках сторов (2026-10-15, коммиты `6711170`, `461ed34`)

### Соцсети в подвале
Строка «Мы в соцсетях» / «Follow ORA» (`.footer__social`, ключ `social_label`) с кнопками и фирменными цветными иконками (SVG прямо в разметке):
- Telegram — https://t.me/ora_artha
- Instagram — https://www.instagram.com/ora_artha/
- Facebook — https://www.facebook.com/profile.php?id=61595192378235

Новую соцсеть добавлять туда же: ссылка с `target="_blank" rel="noopener"`, класс `footer__social-link`, `aria-label` с названием.

### Кнопки сторов
- **Всегда:** кнопка App Store крупнее (60 px), с золотой обводкой и тенью; на телефоне — во всю ширину.
- **Переход из соцсетей.** Ранний скрипт в `<head>` ставит класс `html.from-social`, если есть `?s=` / `utm_source`, реферер Instagram / Facebook / Telegram / Threads или встроенный браузер этих приложений (Instagram, FBAN/FBAV, Telegram в user agent). Признак запоминается в `sessionStorage` (`ora_from_social`) до конца визита. Что меняется: иконка в шапке 160 → 88 px (кнопки попадают в первый экран), кнопка App Store три раза пульсирует (`@keyframes store-pulse`). При `prefers-reduced-motion` пульса нет.
- **Нижняя панель на телефоне** (`#store-bar`, ширина ≤ 720 px, `<script id="store-cta">`). Появляется, когда кнопки в шапке ушли за экран, прячется у подвала. На iPhone — App Store (ссылка получает метку `ct` тем же скриптом `campaign-links`), на Android — «Скоро в Google Play»: прокручивает к шапке и нажимает основную кнопку `#gplay-btn` (тот же счётчик кликов и форма).

### Проверка
С телефона: `www.ora-app.io/?s=ig` — маленькая иконка, пульс App Store; прокрутить вниз — панель снизу; дойти до подвала — панель прячется. Без метки иконка обычного размера, панель при прокрутке та же.
