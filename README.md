# Дизайн-система Яндекс Книг (экспериментальная)

Код-версия дизайн-системы, выгруженная из Figma: **Ядро** (токены) + **Компоненты**.
Чистый CSS custom properties + HTML, без сборщиков и зависимостей — под статические страницы Книг.

> Статус: **в работе**. Источник — Figma Dev Mode MCP (локальный Figma Desktop).
> Готово: токены (Ядро) + компоненты `button`, `toggle`. Остальные категории — по мере обхода.

## Источники в Figma

| Часть      | Файл                        | fileKey                  |
|------------|-----------------------------|--------------------------|
| Ядро       | 🧪 Ядро (экспериментальное)   | `O3n7CfrOYzFA3HaFn8zilB` |
| Компоненты | 🧪 Компоненты (эксперим.)     | `19xJVW6drCREmRWGXnBCeA` |

## Структура

```
tokens/
  tokens.json      # источник правды: все переменные, сгруппированы
  variables.css    # :root { --... } — цвета, отступы, радиусы, тени
  typography.css   # текстовые токены (YS Text) — vars + классы
components/
  <name>/<name>.css, <name>.html   # компоненты через var(--...) из tokens
figma/
  figma.config.json                # fileKeys для будущего Code Connect
reference/                          # скриншоты из Figma для сверки
```

## Использование

```html
<link rel="stylesheet" href="tokens/variables.css">
<link rel="stylesheet" href="tokens/typography.css">
<link rel="stylesheet" href="components/button/button.css">
```

## Обновление при изменениях в Figma

1. Открыть нужный файл в Figma Desktop, выделить фрейм/страницу.
2. Через Dev Mode MCP снять `get_variable_defs` / `get_design_context`.
3. Обновить `tokens.json` → перегенерировать `variables.css` / `typography.css`.

## Соответствие Figma → код

| Figma | Код | Статус |
|-------|-----|--------|
| Ядро → Interface (цвета, темы) | `tokens/variables.css` | ✅ 6 тем + палитра |
| Ядро → Text styles (типографика) | `tokens/typography.css` | ✅ 27 стилей |
| Компоненты → Buttons / `butons` (14:1378) | `components/button/` | ✅ primary/secondary/plain/plus × big/medium/small × states |
| Компоненты → Buttons / `toggle` (23:4080) | `components/toggle/` | ✅ iOS/Android × on/off |
| Ядро → Icons (9973:66041) | `components/icon/` + `reference/icons-catalog.json` | ✅ core-набор 33 SVG (24px, outline/fill); каталог всех 188 |
| Компоненты → Covers, Navigation, Informers, Snippets, Meta, Avatars, Mini player, Section, Native elements | — | ⏳ не выгружено |

### Точность
Кнопки сняты **попиксельно** через `get_design_context` (после включения в Figma Desktop
галочки Dev Mode → MCP → «Allow overwriting files on disk»):
- big: `height 52 · padding 0 24 · font YS Text Medium 16 · letter-spacing 0.16px · radius 1000px`
- medium: `40 · 0 16 · 14 · 0.14px`; small: `32 · 0 16 · 14 · 0.14px` (medium/small делят метрики)
- primary `bg=var(--elements) text=var(--bg)`, secondary `var(--menu-bg)/var(--elements)`,
  plain `transparent/var(--elements)`, plus — точный градиент
  `linear-gradient(90deg,#FF5C4D 0%,#EB469F 26.56%,#8341EF 75%,#3F68F9 100%)`, текст white.
- **Иконки+текст в кнопках (точно из Figma, node 1:460/1:1095/1:1092):**
  big — контейнер иконки 24, gap 8, паддинг 20/24 (иконка/текст-сторона);
  medium/small — контейнер 20, gap 4, паддинг 12/16. Глиф внутри контейнера с инсетом ~15%
  (в SVG нормализован viewBox под 24-канвас по реальным границам → правильный размер и отступы).
  Обычные кнопки — иконка `plus`, plus-тип — `yandex-plus` (Я+ бейдж). Демо подключает `../icon/icon.css`.
  Классы `.btn--left`/`.btn--right` дают асимметрию паддингов (иконка-сторона −4px).

Осталось приблизительным:
- **Ховеры** — подобраны (в Figma отдельных hover-состояний нет).
- Иконки экспортированы «edge-to-edge» (внутренние инсеты Figma не сохранены) — норма для icon-набора.

### Иконки (компонент Icon)
- 33 SVG core-набора выгружены из «Ядра» после включения Allowed-directories-записи
  (галочка «Allow overwriting files on disk»). Метод: один вызов на узел-вектор (id символа + 1) → 1 файл.
- Пост-обработка: `fill` → `currentColor`, `preserveAspectRatio` → `xMidYMid meet`.
- Система на CSS mask (`components/icon/icon.css`): цвет через `color`, размер через класс `.icon-16/20/32/48`.
- Полный инвентарь всех 188 иконок — `reference/icons-catalog.json` (можно доизвлечь по тому же методу).

> **Локальный просмотр:** CSS-маски по `file://` Chrome может блокировать → запусти
> `python3 -m http.server 8080` в корне папки и открой `http://localhost:8080/`.
- **Plus-градиент** — `Plus/primary` в Figma не отдаётся как hex (брендовый градиент); задан по эталонному скриншоту.
- **letterSpacing / lineHeight** — трактованы как % (см. `tokens.json` → `$meta.notes`).
- Отдельной шкалы **отступов/радиусов** в «Ядре» не найдено — геометрия берётся из компонентов.
