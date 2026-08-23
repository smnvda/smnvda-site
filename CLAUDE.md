# smnvda-site

Портфолио-резюме Дмитрия Семенова (бизнес-архитектор), домен **smnvda.ru**.
Astro + Tailwind CSS 4 + Alpine.js, статическая сборка, деплой на GitHub Pages через GitHub Actions.

## Текущее состояние

Одностраничное резюме на базе шаблона [Smilesharks/dev-portfolio](https://github.com/Smilesharks/dev-portfolio), наполненное реальными данными из резюме с hh.ru. Полностью заменило прежнее многостраничное портфолио (Astro + свой CSS, экспорт из Webflow) — та версия сохранилась только в истории git, на сайте её больше нет.

- **Весь контент — в `cv.json`** (формат jsonresume.org). Правки текста, опыта работы, навыков, образования, проектов делаются там, компоненты трогать не нужно.
- Секции сайта — `src/components/sections/*.astro` (Hero, About, Experience, Education, Skills, Projects), каждая читает свои поля из `cv.json` через алиас `@cv`.
- Тема: светлая/тёмная/системная + 5 цветовых схем (`default`, `blue`, `red`, `green`, `cyber`) — переключаются на клиенте (`ThemeSwitch.astro`, `localStorage`), поле `basics.theme` в `cv.json` на переключение не влияет.
- Версия для печати в PDF — через `@media print` стили в компонентах.
- Профиль: контакты — телефон, почта, Telegram (иконка Telegram добавлена отдельно в `src/icons/Telegram.astro` и подключена в `Hero.astro` — в оригинальном шаблоне её не было, только GitHub/LinkedIn/X).
- Фото профиля: `public/smnvda.jpg` (реальное, перенесено из прежней версии сайта).
- Favicon: `public/favicon-16x16.png` / `favicon-32x32.png` (реальные, перенесены из прежней версии сайта) — generic favicon.svg из шаблона удалён.
- Заголовок вкладки браузера — просто `{name}` из `cv.json` (`Layout title={name}` в `index.astro`), без "Portafolio - ...".
- Убрана нижняя атрибуция автора шаблона ("Made by Ariel", `src/components/sections/Tail.astro` удалён) и оранжевая подсказка "Press Cmd+K" (`KeyboardManager.astro` очищен от неё) — командная палитра по Cmd/Ctrl+K продолжает работать, просто без навязчивой плашки.
- Деплой: `.github/workflows/deploy.yml` — при каждом push в `master` собирает Astro (`withastro/action`) и публикует в GitHub Pages (`actions/deploy-pages`). Использует npm (`package-lock.json`) — `pnpm-lock.yaml` из шаблона удалён, он ломал сборку (`withastro/action` выбирал pnpm без указанной версии).
- Кастомный домен: `public/CNAME` (`smnvda.ru`) + `site: "https://smnvda.ru"` в `astro.config.mjs`.
- Репозиторий публичный — приватным GitHub Pages с кастомным доменом не работает без платного плана. Вся история git проверена на секреты/утечки (API-ключи, пароли, `.env`) — ничего не найдено. В `cv.json` осознанно лежат реальные контакты (телефон, почта, Telegram) — это ожидаемо для публичного резюме, не утечка.

## Известные допущения в данных (проверить с пользователем)

- Дата начала обучения в вузе в `cv.json` (`education[0].startDate`, сейчас `2007-09-01`) — в резюме с hh.ru указан только год выпуска (2012), начало оценено как типичная 5-летняя программа. Нужно подтвердить у пользователя точную дату.

## Миграция хостинга: Cloudflare → GitHub Pages + Jino DNS

Сайт переехал с Cloudflare (Pages + DNS) на:
- **Хостинг** — GitHub Pages
- **DNS** — регистратор **Jino.ru** (домен `smnvda.ru`), Cloudflare полностью убран

### Статус на конец сессии
- NS-делегация на уровне регистратора переключена на `ns1-4.jino.ru` (было `khalid.ns.cloudflare.com` / `sharon.ns.cloudflare.com`), статус `DELEGATED` — подтверждено в реестре доменных имён
- A-записи корневого домена в Jino указывают на 4 IP GitHub Pages: `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
- Сайт уже открывался на `http://smnvda.ru` (пропагация DNS частично/полностью прошла на момент проверки пользователем), но **HTTPS ещё не подтверждён** — на скриншоте пользователя браузер показывал "Не защищено". Нужно зайти в Settings → Pages и проверить статус DNS check; если он зелёный — включить **Enforce HTTPS** (GitHub сам выпустит сертификат)
- Если DNS check всё ещё не проходит — смотреть пропагацию на whatsmydns.net (`#NS/smnvda.ru` и `#A/smnvda.ru`), может занимать часы, редко — до суток

### DNS-записи в Jino (⚠️ НЕ трогать почтовые)
В зоне `smnvda.ru` вперемешку лежат записи для сайта и для почты:
- **Не трогать**: `mail.smnvda.ru` (A), `smnvda.ru` (MX → mail.smnvda.ru), `smtp.smnvda.ru` (CNAME → smtp.jino.ru), `autoconfig.smnvda.ru` (CNAME), `_autodiscover._tcp` (SRV), `dkim._domainkey` (TXT) — держат корпоративную почту `@smnvda.ru`
- **Не трогать**: `n8n.smnvda.ru` (A) — отдельный сервис автоматизации, к сайту не относится
- **Не трогать**: `*.smnvda.ru` (A, wildcard на старый сервер) — не влияет на корневой домен, пока не мигрирован

## Дальнейшие шаги (TODO)
1. Проверить и при необходимости поправить дату начала обучения в `cv.json`
2. Проверить статус DNS check в Settings → Pages, включить **Enforce HTTPS**, когда станет доступно
3. Удалить проект в Cloudflare, когда `https://smnvda.ru` заработает стабильно по HTTPS
4. Дальнейшая адаптация сайта — редактировать `cv.json`; при изменении структуры секций — компоненты в `src/components/sections/`
5. На GitHub осталось несколько уже смёрженных feature-веток (`add-claude-md`, `fix-pnpm-lockfile`, `fix-skills-chips`, `polish-layout`, `setup-github-pages`, `unify-skills-chip-style`, а также старая `claude/project-analysis-2zj6io`) — GitHub не удалил их автоматически после мёржа. Можно удалить для порядка, они больше не нужны

## Команды
```bash
npm install
npm run dev      # localhost:4321
npm run build    # astro check + сборка в dist/
```

## История PR (все влиты в master)
- PR #1 — замена демо-контента на реальный контент многостраничного сайта (позже полностью заменено PR #4)
- PR #2 — настройка деплоя на GitHub Pages для smnvda.ru
- PR #3 — добавлен CLAUDE.md
- PR #4 — полная замена сайта на шаблон dev-portfolio с реальным резюме
- PR #5 — фикс деплоя (удалён лишний pnpm-lock.yaml)
- PR #6 — фикс опечатки в Tailwind-классах чипов Skills (не было рамки/заливки)
- PR #7 — убран подвал шаблона (атрибуция + подсказка Cmd+K), восстановлен реальный favicon, упрощён title
- PR #8 — единый стиль чипов между Skills и Experience (цвет текста)
