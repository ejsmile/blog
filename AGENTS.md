# Pavel Karasov Blog - Документация проекта

## О проекте

Личный блог на Hugo с автоматическим деплоем через GitHub Actions.

**URL**: https://p.karasov.net/

## Архитектура

### Два репозитория

1. **ejsmile/blog** - исходники (контент, конфигурация, workflow)
2. **ejsmile/ejsmile.github.io** - собранный сайт (автоматически обновляется)

### Технологии

- **Hugo** - генератор статических сайтов
- **Archie Theme** - минималистичная тема (git submodule от [athul/archie](https://github.com/athul/archie))
- **GitHub Actions** - CI/CD
- **GitHub Pages** - хостинг

## Особенности проекта

### Тема Archie (Git Submodule)

Тема подключена как submodule из внешнего репозитория:

```bash
# Обновление темы
git submodule update --remote themes/archie
```

### Настройки темы (config.toml)

- `mode="auto"` - автопереключение светлой/тёмной темы
- `useCDN=false` - локальные шрифты и иконки
- `pagerSize=4` - 4 поста на страницу

### Кастомный домен

Домен `p.karasov.net` настроен через:
- CNAME запись: `p.karasov.net → ejsmile.github.io`
- `baseURL = 'https://p.karasov.net/'` в config.toml
- Файл `CNAME` создаётся автоматически в workflow

## Локальная разработка

### Клонирование проекта

```bash
git clone --recurse-submodules https://github.com/ejsmile/blog.git
cd blog

# Если забыли --recurse-submodules
git submodule update --init --recursive
```

### Локальный сервер

```bash
# С черновиками
hugo server -D

# Только опубликованные
hugo server
```

Сайт: http://localhost:1313

### Создание поста

```bash
hugo new posts/my-post.md        # английский
hugo new posts/my-post.ru.md     # русский
```

### Локальная сборка

```bash
hugo --minify
# Результат в public/
```

## GitHub Actions CI/CD

### Workflow (.github/workflows/hugo.yml)

**Триггеры:**
- Push в ветку `master`
- Ручной запуск (workflow_dispatch)

**Процесс:**
1. Checkout с submodules
2. Установка Hugo
3. Сборка с минификацией
4. Деплой в `ejsmile/ejsmile.github.io`

### Настройка токена

1. Создать токен: https://github.com/settings/tokens/new (scope: `repo`)
2. Добавить в Settings → Secrets and variables → Actions
3. Name: `PERSONAL_TOKEN`

### Ручной запуск

GitHub → Actions → "Deploy Hugo site to Pages" → Run workflow → Выбрать `master`

## Игнорируемые файлы (.gitignore)

```
/public               # Локальная сборка
/resources            # Кэш Hugo
.hugo_build.lock      # Lock-файл
```

**Важно:** `public/` не коммитится, собирается автоматически в GitHub Actions.

## Структура контента

```
content/
├── about.md           # Страница "О себе"
└── posts/             # Посты
    ├── *.en.md        # Английские
    └── *.ru.md        # Русские
```

## Полезные команды

```bash
hugo config                          # Проверка конфигурации
hugo list all                        # Список всех постов
git submodule update --remote        # Обновление темы
hugo --logLevel debug                # Отладка
```

## Troubleshooting

### Тема не загружается
```bash
git submodule update --init --recursive
```

### Workflow не запускается
- Изменения закоммичены и запушены в `master`?
- Секрет `PERSONAL_TOKEN` добавлен?

### Сайт не обновляется
- Проверить логи: GitHub → Actions
- В `ejsmile.github.io`: Settings → Pages → Source: Deploy from a branch

### Локальный сервер не работает
```bash
hugo version                          # Проверить Hugo
ls themes/archie                      # Проверить тему
git submodule update --init --recursive
```

## Контакты

- GitHub: [@ejsmile](https://github.com/ejsmile)
- Email: pavel@karasov.net
- Telegram: [@PavelKarasov](https://t.me/PavelKarasov)
- LinkedIn: [pavel-karasov](http://www.linkedin.com/in/pavel-karasov)
