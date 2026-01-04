# Настройка Credentials для Workflow

## Проблема: "Credentials not found"

Если вы видите ошибку "Credentials not found" для какого-либо узла, вам нужно настроить соответствующие credentials в n8n.

## Узлы, требующие credentials:

### 1. Generate Cover Image (HuggingFace)1
**Тип:** HTTP Header Auth  
**API:** HuggingFace Inference API

**Как настроить:**
1. Перейдите в Settings → Credentials в n8n
2. Создайте новый credential типа "HTTP Header Auth"
3. Название: "HuggingFace API" (или любое другое)
4. В поле "Name" введите: `Authorization`
5. В поле "Value" введите: `Bearer YOUR_HUGGINGFACE_TOKEN`
6. Получить токен можно на: https://huggingface.co/settings/tokens

**Примечание:** Узел настроен с `continueOnFail: true`, поэтому workflow продолжит работу даже без credentials (изображение просто не будет сгенерировано).

### 2. Groq API узлы
**Тип:** HTTP Header Auth  
**API:** Groq API

**Как настроить:**
1. Создайте credential типа "HTTP Header Auth"
2. Название: "Header Auth account" (или используйте существующий)
3. В поле "Name" введите: `Authorization`
4. В поле "Value" введите: `Bearer YOUR_GROQ_API_KEY`
5. Получить API ключ можно на: https://console.groq.com/keys

### 3. Google Sheets узлы
**Тип:** Google Sheets OAuth2

**Как настроить:**
1. Создайте credential типа "Google Sheets OAuth2 API"
2. Название: "Google Sheets account"
3. Следуйте инструкциям n8n для настройки OAuth2
4. Убедитесь, что у вас есть доступ к Google Sheet с ID: `13vWuIBj6AOUumXpT4-geMwjHhE-eBlAf1_TjKNBduLU`

### 4. WordPress REST API узлы
**Тип:** HTTP Basic Auth

**Как настроить:**
1. Создайте credential типа "HTTP Basic Auth"
2. Название: "WordPress Basic Auth" (или любое другое)
3. Введите ваш WordPress username
4. Введите Application Password (не обычный пароль!)
5. Для создания Application Password в WordPress:
   - Перейдите в Users → Your Profile
   - Прокрутите до "Application Passwords"
   - Создайте новый пароль для n8n

### 5. Google Custom Search API
**Тип:** Встроен в URL (можно вынести в Config)

**Текущая настройка:** API ключ встроен в URL узла "Google Custom Search1"

**Как изменить:**
1. Получите API ключ на: https://console.cloud.google.com/apis/credentials
2. Создайте Custom Search Engine на: https://programmablesearchengine.google.com/
3. Обновите значения в узле "Config":
   - `googleSearchApiKey`: ваш API ключ
   - `googleSearchCx`: ваш Search Engine ID

## Альтернативное решение: Отключить генерацию изображений

Если вы не хотите использовать HuggingFace API, вы можете:

1. **Отключить узел "Generate Cover Image (HuggingFace)1"**:
   - Кликните правой кнопкой на узел
   - Выберите "Disable Node"

2. **Или удалить connection к этому узлу** в узле "Parse Metadata"

3. **Или использовать дефолтное изображение** - измените узел "Create WordPress Post1", чтобы он не требовал `featured_media`

## Проверка credentials

После настройки credentials:
1. Откройте каждый узел, который требует credentials
2. В поле "Authentication" выберите соответствующий credential
3. Сохраните workflow
4. Протестируйте выполнение

## Важно

- Узел "Generate Cover Image (HuggingFace)1" настроен с `continueOnFail: true`, поэтому workflow не остановится, если нет credentials
- Все остальные узлы требуют обязательной настройки credentials для работы
- Храните API ключи в безопасности и не делитесь ими публично

