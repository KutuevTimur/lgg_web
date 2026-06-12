# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**LGG School** — лендинг сайт команды репетиторов по математике (онлайн, вся Россия).

- **Сайт:** https://www.lgg-school.ru/
- **ВК сообщество:** https://vk.com/lggmathnn
- **Запись:** https://vk.me/lggmathnn
- **Цена:** 1 000 ₽/занятие
- **Преподаватели:** Сергей (Лобачевский), Тимур (ВШЭ), Даниил (Лобачевский), по 20 лет, 2 курс
- **Аудитория:** 5–8 класс, ОГЭ (9 кл), ЕГЭ (10–11 кл), пересдача ОГЭ/ЕГЭ

## Стек и архитектура

Проект — **статический одностраничный лендинг** без сборщика и фреймворков:

```
index.html          # Вся страница — единственный файл
img/                # Скриншоты продуктов (LK1-3, stepik, VK бот и др.)
resource/
  plan_9_class.md   # Помесячный план подготовки к ОГЭ (Сент–Май)
.claude/
  skills/ui-ux-pro-max/   # UI/UX скилл с базами стилей, палитр, типографики
```

**Зависимости:**
- Tailwind CSS — **предкомпилирован в статический CSS, встроен инлайном в `<head>`** (НЕ CDN!). Раньше был `cdn.tailwindcss.com`, но это runtime-JS-компилятор → давал FOUC (вспышку нестилизованной страницы) в медленных браузерах (ВК). Убрано.
- Poppins + Inter — Google Fonts с preconnect

**⚠️ Не возвращать `cdn.tailwindcss.com`** — вернётся FOUC. Если меняешь Tailwind-классы в `index.html`, пересобери инлайн-CSS:
```bash
npx tailwindcss@3 -c tailwind.config.js -i input.css -o out.css --minify
# затем заменить содержимое первого <style> в <head> на out.css
```
(`tailwind.config.js` лежит в корне, `content: ['./index.html']`.)

**Деплой:** Vercel (проект `kutuevtimurs-projects/lgg-web`), домен `www.lgg-school.ru`. GitHub `KutuevTimur/lgg_web` подключён — пуш в `main` авто-деплоит. Статика, build не нужен. `.vercelignore` прячет `resource/`, `.claude`, `*.md`, `tailwind.config.js`.

**Предпросмотр:** открыть `index.html` прямо в браузере или использовать любой локальный HTTP-сервер:
```bash
python3 -m http.server 8080
```

## CSS-система (всё в `<style>` внутри index.html)

**CSS-переменные (корень темы):**
```css
--green-btn: #16A34A   /* Основной зелёный — кнопки, акценты */
--green-light: #EDF7EC /* Светло-зелёный фон */
--blue: #0056D2        /* Синий — вторичный акцент */
--bg: #F8FAF7          /* Фон страницы (тёплый офф-вайт) */
--surface: #FFFFFF     /* Фон карточек */
--border: #E4E7E3      /* Граница элементов */
--border-strong: #C8CFCC
--text: #111A10        /* Основной текст */
--muted: #5A6B57       /* Приглушённый текст */
--faint: #9BAA97       /* Очень бледный текст (stag, подписи) */
```

**Ключевые CSS-классы:**
- `.h` — заголовочный шрифт (Poppins 800, tight tracking)
- `.stag` / `.stag-g` — секционный лейбл сверху (uppercase, мелкий)
- `.div-line` — горизонтальный разделитель между секциями
- `.fi` / `.fi.on` — fade-in анимация при скролле (IntersectionObserver)
- `.card` / `.tcard` / `.tecard` / `.rcard` / `.mcard` — разные типы карточек
- `.btn-g` — основная зелёная кнопка
- `.btn-o` — вторичная outline кнопка
- `.plan-tab` / `.plan-tab.active` — вкладки выбора плана
- `.plan-body` / `.hidden` — контент вкладок (toggle visibility)
- `.overlay` / `.overlay.active` — полноэкранные оверлеи продуктов
- `.ov-close` — кнопка закрытия оверлея
- `.fbar` — фиксированная нижняя панель с CTA

**Важно по перформансу:**
- `hover:` эффекты завёрнуты в `@media (hover: hover)` — не применяются на тачскрине
- `backdrop-filter` только на desktop (md+), не на мобилке
- `prefers-reduced-motion` отключает все анимации

## JavaScript (инлайн в конце index.html)

Три функциональных блока:

**1. Fade-in при скролле:**
```js
IntersectionObserver — unobserve после первого показа, rootMargin -40px снизу
```

**2. Переключение планов:**
```js
switchPlan(id)  // 'p58' | 'p9' | 'p10' | 'p11' | 'retake'
// Скрывает все .plan-body, показывает нужный, переключает .active на табах
```

**3. Оверлеи продуктов:**
```js
openOv(id)   // 'buildin' | 'stepik' | 'bot' | 'vk'
closeOv(id)
// body.ov-open → масштабирует #page, overlay.active → показывает оверлей
// Свайп вниз на мобилке закрывает оверлей
```

## Структура секций (порядок на странице)

1. **Hero** — заголовок + рейтинг + цена + CTA
2. **Stats** — 40+ учеников, 2 года, 5.0, 3 препода
3. **Why Us** — 4 карточки преимуществ
4. **Твой план** — 5 вкладок с помесячными планами (p58, p9, p10, p11, retake)
5. **Экосистема** — 4 карточки-триггера для оверлеев
6. **Преподаватели** — 3 карточки (Сергей, Тимур, Даниил)
7. **Отзывы** — рейтинг 5.0 + 3 отзыва (на светло-зелёном фоне)
8. **Реферальная программа** — 3 шага, 1 000 ₽ за друга после 3 уроков
9. **Цена** — 1 000 ₽/занятие + список включённого
10. **Финальный CTA**
11. **Оверлеи** (Buildin, Stepik, VK Бот, VK комьюнити) — вне потока, position fixed

## Контент который нельзя менять

Следующее отражает **реальную инфраструктуру** — не выдумывать, не заменять:
- Скриншоты в `img/` — реальные скрины продуктов (Buildin ЛК, Stepik, VK бот)
- Имена и вузы преподавателей
- Ссылки на VK (vk.com/lggmathnn, vk.me/lggmathnn)
- Цена 1 000 ₽/занятие
- Рейтинг 5.0
- Реферальная программа: 1 000 ₽ на карту после 3 уроков друга

## UI/UX Pro Max скилл

Установлен в `.claude/skills/ui-ux-pro-max/`. Подгружается автоматически.

Данные для дизайн-решений:
- `.claude/skills/ui-ux-pro-max/data/colors.csv` — палитры по типу продукта
- `.claude/skills/ui-ux-pro-max/data/styles.csv` — 67 UI-стилей
- `.claude/skills/ui-ux-pro-max/data/typography.csv` — шрифтовые пары
- `.claude/skills/ui-ux-pro-max/data/landing.csv` — паттерны лендингов
- `.claude/skills/ui-ux-pro-max/data/ux-guidelines.csv` — UX правила

Переустановить скилл: `npx uipro-cli init --ai claude`

## Планы обучения (resource/)

`resource/plan_9_class.md` — детальный помесячный план ОГЭ (Сент–Май) с номерами заданий.
Планы для 10, 11 класса и пересдачи — реализованы прямо в HTML секции `#plan-p10`, `#plan-p11`, `#plan-retake`.
