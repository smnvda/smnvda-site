# smnvda-site

Портфолио-сайт Дмитрия Семенова (бизнес-архитектор), домен **smnvda.ru**.
Astro, статическая сборка, деплой на GitHub Pages через GitHub Actions.

## Текущее состояние веток

### `master` — боевая версия, задеплоена
Многостраничное портфолио на Astro с собственным CSS (экспорт из Webflow).

- Весь контент (профиль, hero, разделы, карточки, контакты) — в `src/data/content.json`. Правки текста делаются там, компоненты трогать не нужно.
- Страницы разделов (`/processes-bpm`, `/systems-integrations`, `/automation`, `/community`) генерируются одним динамическим роутом `src/pages/[slug].astro` из `content.json` — не создавайте под них отдельные файлы.
- `src/components/Sidebar.astro` — навигация, тоже рендерится из `content.json`.
- `/contacts` — отдельная страница с плейсхолдерами (Telegram/телефон/почта/hh.ru) в `content.json.contacts`, ждут реальных данных.
- Деплой: `.github/workflows/deploy.yml` — при каждом push в `master` собирает Astro (`withastro/action`) и публикует в GitHub Pages (`actions/deploy-pages`).
- Кастомный домен: `public/CNAME` (`smnvda.ru`) + `site: 'https://smnvda.ru'` в `astro.config.mjs`.
- Репозиторий переведён в **публичный** — приватным GitHub Pages с кастомным доменом не работает без платного плана.

### `claude/project-analysis-2zj6io` — черновик, НЕ влит
Полная замена сайта на шаблон [Smilesharks/dev-portfolio](https://github.com/Smilesharks/dev-portfolio): Astro + Tailwind CSS 4 + Alpine.js, одностраничное резюме в формате jsonresume.org (`cv.json`), светлая/тёмная/системная тема, 5 цветовых схем, версия для печати в PDF.

**Сейчас в ветке демо-данные шаблона, не реальный контент.** Решение — переходить ли на этот шаблон вместо текущего `master` — ещё не принято. Если решите переходить: нужно перенести реальный контент из `content.json` (master) в `cv.json` формата jsonresume.org.

## Миграция хостинга: Cloudflare → GitHub Pages + Jino DNS

Раньше сайт был на Cloudflare (Pages + DNS). Переезжаем на:
- **Хостинг** — GitHub Pages (готово, см. `master`)
- **DNS** — обратно на регистратора **Jino.ru** (домен `smnvda.ru`), убираем Cloudflare полностью

### Статус на 2026-08-23
- NS-делегация на уровне регистратора уже переключена: `ns1-4.jino.ru` вместо `khalid.ns.cloudflare.com` / `sharon.ns.cloudflare.com`, статус `DELEGATED` — подтверждено в реестре доменных имён
- Пропагация NS по миру ещё идёт (whatsmydns.net на момент проверки показывал старые cloudflare NS почти везде) — GitHub Pages выдавал `NotServedByPagesError`, это ожидаемо и требует времени (часы, иногда до 24–48ч для NS на `.ru`)
- Настройки со стороны GitHub и DNS-записи уже верны, дальше — просто ждать пропагации

### DNS-записи в Jino (⚠️ НЕ трогать почтовые)
В зоне `smnvda.ru` вперемешку лежат записи для сайта и для почты:
- **Не трогать**: `mail.smnvda.ru` (A), `smnvda.ru` (MX → mail.smnvda.ru), `smtp.smnvda.ru` (CNAME → smtp.jino.ru), `autoconfig.smnvda.ru` (CNAME), `_autodiscover._tcp` (SRV), `dkim._domainkey` (TXT) — вся эта пятёрка держит корпоративную почту `@smnvda.ru`, удаление сломает приём/отправку писем
- **Не трогать**: `n8n.smnvda.ru` (A) — отдельный сервис автоматизации, к сайту не относится
- **Не трогать**: `*.smnvda.ru` (A, wildcard на старый сервер) — не влияет на корневой домен, но пока не мигрирован
- **Сайт**: корневой `smnvda.ru` (A) теперь указывает на 4 IP GitHub Pages — `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`

## Дальнейшие шаги (TODO)
1. Дождаться полной пропагации NS (проверять на whatsmydns.net/#NS/smnvda.ru) — пользователь мониторит сам
2. Когда в GitHub Settings → Pages статус "DNS check" станет успешным — включить **Enforce HTTPS**
3. Убедиться, что `https://smnvda.ru` работает, и после этого снести проект/страницу в Cloudflare — она станет не нужна
4. Решить судьбу ветки `claude/project-analysis-2zj6io` (шаблон dev-portfolio): доработать и влить, или отказаться
5. Дальнейшая адаптация сайта — предположительно продолжится на `master`, редактируя `src/data/content.json` и, при необходимости, компоненты в `src/components/`

## Команды
```bash
npm install
npm run dev      # localhost:4321
npm run build    # сборка в dist/
```

## История PR
- PR #1 — замена демо-контента шаблона на реальный контент (влит в master)
- PR #2 — настройка деплоя на GitHub Pages для smnvda.ru (влит в master)
