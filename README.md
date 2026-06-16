# RAG-агент для академии селлеров Lamoda

Тестовое задание: реализовать агента, который отвечает на вопросы продавцов по базе знаний [academy.lamoda.ru](https://academy.lamoda.ru).

## Пайплайн

```
Парсинг academy.lamoda.ru → Чанкование → Гибридный ретривер → ReAct-агент
```

1. **Парсер** — Playwright обходит сайт (Bitrix CMS, server-rendered), собирает статьи за два прохода: сначала разделы с главной, потом статьи внутри каждого раздела. HTML конвертируется в Markdown.

2. **Чанкование** — статьи режутся на куски по ~400 слов с перекрытием 50 слов. Каждый чанк хранит `source` (URL) и `title`.

3. **Гибридный ретривер** — BM25 + dense embeddings (`intfloat/multilingual-e5-small`), слияние через RRF (Reciprocal Rank Fusion). BM25 хорошо работает по точным терминам (FBS, SKU, API), dense — по смыслу.

4. **ReAct-агент** — цикл `THOUGHT → ACTION → OBSERVATION → FINAL ANSWER` реализован руками без фреймворков. LLM вызывает `search_knowledge_base(query)`, получает результаты и формулирует ответ со ссылками на источники.

## Запуск

Ноутбук рассчитан на Google Colab, запускать ячейки сверху вниз.

```
Runtime → Run all
```

Нужен OpenRouter API-ключ — вставить в ячейку 5 (`OPENROUTER_API_KEY`). По умолчанию используется `openrouter/owl-alpha` (роутинг по бесплатным провайдерам) с запасными моделями.

## Стек

| Компонент | Библиотека |
|-----------|------------|
| Парсинг | `playwright`, `beautifulsoup4`, `markdownify` |
| BM25 | `rank_bm25` |
| Embeddings | `sentence-transformers` (`multilingual-e5-small`) |
| LLM | OpenRouter API (OpenAI-compatible) |

## Примеры вопросов

- Как зарегистрироваться на Lamoda как продавец?
- В чём разница между FBS и FBO?
- Как запустить таргетированные акции?
- Какие требования для фотографий часов?
- Что должно быть на этикетке обуви?
