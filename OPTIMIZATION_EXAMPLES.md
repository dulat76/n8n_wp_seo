# Примеры кода для оптимизаций

## 1. Объединенный запрос для метаданных

### Текущий подход (3 запроса):
```json
// Create Slug (Groq)1
{
  "model": "llama-3.3-70b-versatile",
  "messages": [{
    "role": "user",
    "content": "Create a URL slug..."
  }],
  "max_tokens": 50
}

// Create Title (Groq)1
{
  "model": "llama-3.3-70b-versatile",
  "messages": [{
    "role": "user",
    "content": "Extract the blog post title..."
  }],
  "max_tokens": 100
}

// Create Meta Description (Groq)1
{
  "model": "llama-3.3-70b-versatile",
  "messages": [{
    "role": "user",
    "content": "Create SEO meta description..."
  }],
  "max_tokens": 150
}
```

### Оптимизированный подход (1 запрос):
```json
{
  "model": "llama-3.3-70b-versatile",
  "messages": [{
    "role": "system",
    "content": "You are an SEO expert. Always respond in valid JSON format."
  }, {
    "role": "user",
    "content": "Create SEO metadata for this blog post. Primary keyword: {{ $('Merge Blog Versions1').first().json.primaryKeyword }}\n\nBlog post:\n{{ $('Merge Blog Versions1').first().json.finalBlogText }}\n\nOutput JSON with:\n- slug: URL slug in English, max 5 words, format: word-word-word\n- title: Title in Russian, max 65 characters, must include primary keyword\n- metaDescription: Meta description in Russian, 150-160 characters, engaging and descriptive\n\nOutput ONLY valid JSON, no other text."
  }],
  "temperature": 0.3,
  "max_tokens": 200,
  "response_format": { "type": "json_object" }
}
```

### Код для парсинга ответа:
```javascript
// В узле Code после объединенного запроса
const response = $input.first().json.choices[0].message.content;
let metadata;

try {
  // Удаляем markdown code blocks если есть
  const cleaned = response.replace(/```json\n?/g, '').replace(/```\n?/g, '').trim();
  metadata = JSON.parse(cleaned);
} catch (error) {
  // Fallback: попытка извлечь JSON из текста
  const jsonMatch = response.match(/\{[\s\S]*\}/);
  if (jsonMatch) {
    metadata = JSON.parse(jsonMatch[0]);
  } else {
    throw new Error('Failed to parse metadata JSON');
  }
}

return [{
  json: {
    slug: metadata.slug || '',
    title: metadata.title || '',
    metaDescription: metadata.metaDescription || '',
    // Сохраняем остальные данные
    finalBlogText: $('Merge Blog Versions1').first().json.finalBlogText,
    keywords: $('Merge Blog Versions1').first().json.keywords,
    primaryKeyword: $('Merge Blog Versions1').first().json.primaryKeyword,
    rowNumber: $('Merge Blog Versions1').first().json.rowNumber
  }
}];
```

## 2. Параллелизация Google Custom Search

### Текущий подход (последовательно):
```
Split Keywords1 → Google Custom Search1 (keyword1) → Google Custom Search1 (keyword2) → ...
```

### Оптимизированный подход:
Используйте узел **Split In Batches** с настройкой:
- Batch Size: 5 (или количество ключевых слов)
- Options: Process all batches in parallel

Или используйте **Code** узел для параллельного выполнения:

```javascript
// В узле Split Keywords1 - оставить как есть
// Но в Google Custom Search1 настроить:
// Options → Continue On Fail: true
// Options → Response: Full Response

// В Aggregate Research1 добавить обработку ошибок:
const items = $input.all();
const allResults = [];

for (const item of items) {
  // Проверка на ошибки
  if (item.error) {
    console.log(`Error for query: ${item.json.query}`, item.error);
    continue;
  }
  
  if (item.json.items && Array.isArray(item.json.items)) {
    for (const result of item.json.items) {
      allResults.push({
        title: result.title,
        link: result.link,
        snippet: result.snippet
      });
    }
  }
}

// Остальной код без изменений
const citations = allResults.map(r => r.link);
const researchText = allResults.map((r, i) => 
  `[${i + 1}] ${r.title}: ${r.snippet}`
).join('\n\n');

const keywordData = $('Get Keywords from Sheet1').first().json;

return [{
  json: {
    citations: citations,
    researchFindings: researchText,
    keywords: keywordData['Ключевые слова'],
    intent: keywordData['Намерение'],
    primaryKeyword: keywordData['Основное ключевое слово'],
    rowNumber: keywordData.row_number
  }
}];
```

## 3. Параллельная генерация HTML и изображения

### Изменение connections в workflow:

**Текущая структура:**
```
Merge Blog Versions1 → Create Slug/Title/Meta → Create HTML → Generate Image → Upload Image
```

**Оптимизированная структура:**
```
Merge Blog Versions1 → Create Slug/Title/Meta → [Create HTML, Generate Image] (параллельно) → Merge Results → Upload Image
```

Добавить узел **Merge** после Create HTML и Generate Image:
```javascript
// Merge HTML and Image
const htmlData = $('Create HTML (Groq)1').first().json;
const imageData = $('Generate Cover Image (HuggingFace)1').first().json;

return [{
  json: {
    html: htmlData.choices[0].message.content,
    image: imageData,
    slug: $('Create Slug (Groq)1').first().json.choices[0].message.content.trim(),
    title: $('Create Title (Groq)1').first().json.choices[0].message.content.trim(),
    metaDescription: $('Create Meta Description (Groq)1').first().json.choices[0].message.content.trim(),
    // остальные данные
  }
}];
```

## 4. Вынос конфигурации в переменные

### Создать узел "Config" в начале workflow:

```javascript
// Config Node (Code)
return [{
  json: {
    wordpressSite: process.env.WORDPRESS_SITE || 'YOUR_WORDPRESS_SITE.com',
    telegramLink: process.env.TELEGRAM_LINK || 'YOUR_TELEGRAM_LINK',
    bannerUrl: process.env.BANNER_URL || 'YOUR_BANNER_URL',
    googleSearchApiKey: process.env.GOOGLE_SEARCH_API_KEY || 'AIzaSyDzLkmLdpw7dK4qDg2ib-q39586mDNiWss',
    googleSearchCx: process.env.GOOGLE_SEARCH_CX || 'c7cc77fb391b648f3'
  }
}];
```

### Использование в других узлах:
```javascript
// В Google Custom Search1
const config = $('Config').first().json;
const url = `https://www.googleapis.com/customsearch/v1?key=${config.googleSearchApiKey}&cx=${config.googleSearchCx}&q={{ $json.query }}`;

// В Create HTML (Groq)1
const config = $('Config').first().json;
const bannerHtml = `<a href='${config.telegramLink}'><img src='${config.bannerUrl}' alt='Telegram' /></a>`;

// В Create WordPress Post1
const config = $('Config').first().json;
const url = `https://${config.wordpressSite}/wp-json/wp/v2/posts`;
```

## 5. Обработка ошибок

### Добавить Error Trigger узлы:

1. **Error Trigger** после каждого критического API вызова
2. **IF** узел для проверки успешности:
```javascript
// Check API Response
const response = $input.first().json;

if (response.error || !response.choices || !response.choices[0]) {
  return [{ json: { hasError: true, error: response.error || 'Unknown error' } }];
}

return [{ json: { hasError: false, data: response } }];
```

3. **Switch** узел для маршрутизации:
   - True → Retry или Fallback
   - False → Продолжить

### Retry логика:
```javascript
// Retry Logic (Code)
const maxRetries = 3;
const currentRetry = $('Retry Counter').first()?.json?.count || 0;

if (currentRetry < maxRetries) {
  return [{
    json: {
      shouldRetry: true,
      retryCount: currentRetry + 1
    }
  }];
}

return [{
  json: {
    shouldRetry: false,
    error: 'Max retries exceeded'
  }
}];
```

## 6. Оптимизация промпта для изображения

### Текущий промпт (статический):
```json
{
  "inputs": "Cute cartoon businessman and robot talking, ice princess theme, magical winter scene, professional blog cover art, 3:4 aspect ratio, no text"
}
```

### Оптимизированный промпт (динамический):
```javascript
// Generate Image Prompt (Code)
const blogData = $('Merge Blog Versions1').first().json;
const primaryKeyword = blogData.primaryKeyword;
const title = $('Create Title (Groq)1').first().json.choices[0].message.content.trim();

// Генерируем промпт на основе темы
const imagePrompt = `Professional blog cover art for article about ${primaryKeyword}, modern minimalist design, clean background, 3:4 aspect ratio, no text, no watermark, high quality, digital art style`;

return [{
  json: {
    imagePrompt: imagePrompt,
    // остальные данные
  }
}];
```

### Использование в Generate Cover Image:
```json
{
  "inputs": "={{ $json.imagePrompt }}",
  "parameters": {
    "negative_prompt": "text, watermark, signature, letters, words, low quality, blurry"
  }
}
```

## 7. Валидация данных

### Узел Validate Data (Code):
```javascript
const data = $input.first().json;

const errors = [];

// Валидация title
if (!data.title || data.title.length > 65) {
  errors.push('Title must be present and max 65 characters');
}

// Валидация meta description
if (!data.metaDescription || data.metaDescription.length < 150 || data.metaDescription.length > 160) {
  errors.push('Meta description must be 150-160 characters');
}

// Валидация slug
if (!data.slug || !/^[a-z0-9-]+$/.test(data.slug)) {
  errors.push('Slug must be lowercase alphanumeric with hyphens');
}

// Валидация контента
if (!data.finalBlogText || data.finalBlogText.length < 2000) {
  errors.push('Blog post must be at least 2000 characters');
}

if (errors.length > 0) {
  return [{
    json: {
      isValid: false,
      errors: errors,
      data: data
    }
  }];
}

return [{
  json: {
    isValid: true,
    data: data
  }
}];
```

## 8. Использование легких моделей для простых задач

### Для slug и title использовать llama-3.1-8b:
```json
{
  "model": "llama-3.1-8b-instant",  // Вместо llama-3.3-70b-versatile
  "messages": [...],
  "max_tokens": 50,
  "temperature": 0.3
}
```

**Экономия:** 
- llama-3.1-8b: ~$0.05 за 1M токенов
- llama-3.3-70b: ~$0.59 за 1M токенов
- **Экономия: ~90% для простых задач**

## 9. Кэширование существующих постов

### Узел Cache Posts (Code):
```javascript
// Проверяем, есть ли кэш
const cacheKey = 'existing_posts_cache';
const cacheTime = 3600000; // 1 час в миллисекундах
const cached = $workflow.staticData[cacheKey];

if (cached && (Date.now() - cached.timestamp < cacheTime)) {
  // Используем кэш
  return [{
    json: {
      posts: cached.data,
      fromCache: true
    }
  }];
}

// Загружаем из Sheets
const posts = $('Get Existing Posts1').all().map(item => item.json);

// Сохраняем в кэш
$workflow.staticData[cacheKey] = {
  data: posts,
  timestamp: Date.now()
};

return [{
  json: {
    posts: posts,
    fromCache: false
  }
}];
```

### Обновление кэша после публикации:
```javascript
// В Update Google Sheet1 добавить обновление кэша
const newPost = {
  'Ссылка': $('Create WordPress Post1').first().json.link,
  'Содержание': $('Create Summary (Groq)1').first().json.choices[0].message.content
};

const cacheKey = 'existing_posts_cache';
const cached = $workflow.staticData[cacheKey];
if (cached) {
  cached.data.push(newPost);
  cached.timestamp = Date.now();
}
```

