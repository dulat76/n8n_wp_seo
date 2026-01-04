# WP SEO Autoposting - Free APIs

Автоматизированный workflow для n8n, который создает и публикует SEO-оптимизированные блог-посты в WordPress, используя только бесплатные API.

## Описание

Этот workflow автоматизирует весь процесс создания SEO-статей:
- Получение ключевых слов из Google Sheets
- Исследование темы через Google Custom Search
- Создание детального плана статьи
- Написание полноценной статьи (2000-2500 слов)
- Добавление внутренних ссылок
- Генерация SEO-метаданных (title, description, slug)
- Создание обложки через AI
- Публикация в WordPress
- Обновление статуса в Google Sheets

## Используемые API

- **Groq API** (Llama 3.3 70B) - генерация контента
- **Google Custom Search API** - поиск источников
- **Google Sheets API** - управление ключевыми словами
- **HuggingFace API** (Stable Diffusion XL) - генерация изображений
- **WordPress REST API** - публикация постов

## Настройка

### 1. Учетные данные

Настройте следующие учетные данные в n8n:
- Google Sheets OAuth2
- HTTP Header Auth (для Groq API)
- HTTP Header Auth (для HuggingFace API)
- HTTP Basic Auth (для WordPress REST API)

### 2. Конфигурация

Перед использованием замените в workflow:
- `YOUR_WORDPRESS_SITE.com` - на ваш домен WordPress
- `YOUR_TELEGRAM_LINK` - на ссылку вашего Telegram канала
- `YOUR_BANNER_URL` - на URL баннера

### 3. Google Sheets

Создайте Google Sheet со следующими колонками:
- Ключевые слова
- Намерение
- Основное ключевое слово
- Уже написана статья?
- Ссылка
- Содержание

## Установка

1. Импортируйте файл `WP SEO Autoposting - Free APIs.json` в n8n
2. Настройте все учетные данные
3. Обновите URL и ссылки в workflow
4. Активируйте workflow

## Использование

Workflow запускается по расписанию (Schedule Trigger) и автоматически:
1. Находит неопубликованные статьи в Google Sheets
2. Создает и публикует статью
3. Обновляет статус в Google Sheets

## Особенности

- ✅ Полностью автоматизированный процесс
- ✅ SEO-оптимизация (ключевые слова, мета-теги, внутренние ссылки)
- ✅ Поддержка русского языка
- ✅ Генерация изображений через AI
- ✅ Использование только бесплатных API

## Лицензия

MIT

