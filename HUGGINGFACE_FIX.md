# Решение проблемы с HuggingFace API

## Проблема: "api-inference.huggingface.co is no longer supported"

**Важно:** HuggingFace прекратил поддержку старого endpoint. Обязательно используйте `router.huggingface.co`.

## Проблема: "The resource you are requesting could not be found"

Эта ошибка может возникать по нескольким причинам:

### 1. Модель недоступна или требует авторизации

**Решение:**
- Убедитесь, что у вас есть валидный HuggingFace API токен
- Некоторые модели требуют принятия лицензии на странице модели
- Перейдите на https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0 и примите лицензию, если требуется

### 2. Неправильный формат URL

**Варианты URL для попытки:**

1. **Старый endpoint (может работать):**
   ```
   https://api-inference.huggingface.co/models/stabilityai/stable-diffusion-xl-base-1.0
   ```

2. **Новый router endpoint:**
   ```
   https://router.huggingface.co/models/stabilityai/stable-diffusion-xl-base-1.0
   ```

3. **С путем /inference:**
   ```
   https://api-inference.huggingface.co/models/stabilityai/stable-diffusion-xl-base-1.0/inference
   ```

### 3. Настройка Credentials

**Требуется:**
- HTTP Header Auth
- Name: `Authorization`
- Value: `Bearer YOUR_HUGGINGFACE_TOKEN`
- Получить токен: https://huggingface.co/settings/tokens

### 4. Альтернативные решения

#### Вариант A: Отключить генерацию изображений

1. Отключите узел "Generate Cover Image (HuggingFace)1"
2. Workflow продолжит работу без изображения
3. Можно использовать дефолтное изображение WordPress

#### Вариант B: Использовать другой бесплатный сервис

**Replicate API (альтернатива):**
- URL: `https://api.replicate.com/v1/predictions`
- Требует API ключ (есть бесплатный tier)
- Более стабильный для генерации изображений

**Stable Diffusion через другую модель:**
- Попробуйте другую модель: `runwayml/stable-diffusion-v1-5`
- Или: `CompVis/stable-diffusion-v1-4`

#### Вариант C: Использовать готовые изображения

1. Создайте коллекцию изображений-заглушек
2. Используйте узел для случайного выбора изображения
3. Загружайте выбранное изображение в WordPress

### 5. Проверка доступности модели

Выполните тестовый запрос через curl:

```bash
curl https://api-inference.huggingface.co/models/stabilityai/stable-diffusion-xl-base-1.0 \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"inputs": "a beautiful landscape"}'
```

Если получаете ошибку 404, модель может быть недоступна или требует другого endpoint.

### 6. Обновление workflow

Текущий workflow настроен с `continueOnFail: true`, поэтому:
- ✅ Workflow не остановится при ошибке генерации изображения
- ✅ Статья будет опубликована без обложки
- ✅ Можно добавить дефолтное изображение позже

### Рекомендация

Если генерация изображений не критична:
1. Оставьте узел отключенным
2. Используйте дефолтное изображение в WordPress
3. Или загружайте изображения вручную

Если генерация критична:
1. Настройте валидный HuggingFace токен
2. Примите лицензию модели на сайте HuggingFace
3. Попробуйте альтернативные модели или сервисы

