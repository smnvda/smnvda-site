# smnvda-site

Портфолио-резюме Дмитрия Семенова (бизнес-архитектор), домен **smnvda.ru**.
Astro + Tailwind CSS 4 + Alpine.js, статическая сборка, деплой на GitHub Pages через GitHub Actions.

## Текущее состояние

Одностраничное резюме на базе шаблона [Smilesharks/dev-portfolio](https://github.com/Smilesharks/dev-portfolio), наполненное реальными данными (заменяет прежнее многостраничное портфолио на Webflow-CSS).

- **Весь контент — в `cv.json`** (формат jsonresume.org). Правки текста, опыта работы, навыков, образования, проектов делаются там, компоненты трогать не нужно.
- Секции сайта — `src/components/sections/*.astro` (Hero, About, Experience, Education, Skills, Projects, Tail), каждая читает свои поля из `cv.json` через алиас `@cv`.
- Тема: светлая/тёмная/системная + 5 цветовых схем (`default`, `blue`, `red`, `green`, `cyber`) — переключаются на клиенте (`ThemeSwitch.astro`, `localStorage`), поле `basics.theme` в `cv.json` на переключение не влияет.
- Версия для печати в PDF — через `@media print` стили в компонентах.
- Профиль: контакты — телефон, почта, Telegram (иконка Telegram добавлена отдельно в `src/icons/Telegram.astro` и подключена в `Hero.astro` — в оригинальном шаблоне её не было, только GitHub/LinkedIn/X).
- Фото профиля: `public/smnvda.jpg` (реальное, перенесено из прежней версии сайта).
- Деплой: `.github/workflows/deploy.yml` — при каждом push в `master` собирает Astro (`withastro/action`) и публикует в GitHub Pages (`actions/deploy-pages`).
- Кастомный домен: `public/CNAME` (`smnvda.ru`) + `site: "https://smnvda.ru"` в `astro.config.mjs`.
- Репозиторий публичный — приватным GitHub Pages с кастомным доменом не работает без платного плана.

## Известные допущения в данных (проверить с пользователем)

- Дата начала обучения в вузе в `cv.json` (`education[0].startDate`, сейчас `2007-09-01`) — в резюме с hh.ru указан только год выпуска (2012), начало оценено как типичная 5-летняя программа. Нужно подтвердить у пользователя точную дату.

## Миграция хостинга: Cloudflare → GitHub Pages + Jino DNS

Сайт переехал с Cloudflare (Pages + DNS) на:
- **Хостинг** — GitHub Pages
- **DNS** — регистратор **Jino.ru** (домен `smnvda.ru`), Cloudflare полностью убран

### Статус
- NS-делегация на уровне регистратора переключена на `ns1-4.jino.ru`, статус `DELEGATED`
- A-записи корневого домена указывают на 4 IP GitHub Pages: `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
- На момент последней проверки сайт уже открывался на `http://smnvda.ru` (пропагация DNS частично прошла), но GitHub Pages DNS check иногда ещё не проходил из-за неполной пропагации по миру — обычно требует времени, проверять на whatsmydns.net

### DNS-записи в Jino (⚠️ НЕ трогать почтовые)
В зоне `smnvda.ru` вперемешку лежат записи для сайта и для почты:
- **Не трогать**: `mail.smnvda.ru` (A), `smnvda.ru` (MX → mail.smnvda.ru), `smtp.smnvda.ru` (CNAME → smtp.jino.ru), `autoconfig.smnvda.ru` (CNAME), `_autodiscover._tcp` (SRV), `dkim._domainkey` (TXT) — держат корпоративную почту `@smnvda.ru`
- **Не трогать**: `n8n.smnvda.ru` (A) — отдельный сервис автоматизации, к сайту не относится
- **Не трогать**: `*.smnvda.ru` (A, wildcard на старый сервер) — не влияет на корневой домен

## Дальнейшие шаги (TODO)
1. Проверить и при необходимости поправить дату начала обучения в `cv.json`
2. Дождаться полной пропагации DNS, включить **Enforce HTTPS** в Settings → Pages, когда DNS check станет успешным
3. Удалить проект в Cloudflare, когда `https://smnvda.ru` заработает стабильно
4. Дальнейшая адаптация сайта — редактировать `cv.json`; при изменении структуры секций — компоненты в `src/components/sections/`

## Команды
```bash
npm install
npm run dev      # localhost:4321
npm run build    # astro check + сборка в dist/
```

## История PR
- PR #1 — замена демо-контента на реальный контент многостраничного сайта (влит в master, позже заменено)
- PR #2 — настройка деплоя на GitHub Pages для smnvda.ru (влит в master)
- PR #3 — добавлен CLAUDE.md (влит в master)
- Полная замена сайта на шаблон dev-portfolio с реальным резюме — ветка `claude/project-analysis-2zj6io`
