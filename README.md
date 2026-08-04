# Custom Codex Profile for Yandex AI Studio (DeepSeek)

## Что и куда класть

Профиль для Codex — это TOML-файл, который содержит настройки модели, провайдера и
дополнительные опции.

```
~/.codex/
├── yandex-ai-studio.config.toml   ← профиль
└── yandex_ai_studio_catalog.json  ← описание модели (опционально)
```

**Запуск:**

```bash
codex --profile yandex-ai-studio
```

Флаг `--profile` автоматически ищет файл `~/.codex/<имя>.config.toml`.

---

## Полный пример конфига

**Файл `~/.codex/yandex-ai-studio.config.toml`:**

```toml
model = "gpt://<folder-id>/deepseek-v4-flash/latest"
model_provider = "yc"
model_catalog_json = "~/.codex/yandex_ai_studio_catalog.json"

# Reasoning для DeepSeek — обязательно должно быть включено!
model_supports_reasoning_summaries = true
model_reasoning_effort = "medium"
model_reasoning_summary = "auto"

web_search = "live"

developer_instructions = """
Do NOT use the apply_patch tool — it is not available in this environment.
Use shell commands (cat, sed, cp, tee, heredocs) or write files directly to make changes.
"""

[tools.web_search]
context_size = "medium"
allowed_domains = ["aistudio.yandex.ru", "https://github.com/yandex-cloud/mcp", "https://yandex.cloud/ru/docs"]

[model_providers.yc]
name = "Yandex Cloud"
base_url = "https://ai.api.cloud.yandex.net/v1"
wire_api = "responses"
env_key = "YC_API_KEY"
env_key_instructions = """
Get your API key from https://aistudio.yandex.ru/
"""
request_max_retries = 8
stream_max_retries = 20
stream_idle_timeout_ms = 900000
requires_openai_auth = false
supports_websockets = false
env_http_headers = { "x-folder-id" = "YC_FOLDER_ID", "OpenAI-Project" = "YC_FOLDER_ID" }

[tui]
status_line = ["model-name", "context-used"]
status_line_use_colors = true
```

---

## Параметры

### `model` — строка подключения

Формат для DeepSeek через Yandex Cloud:

```
gpt://<folder-id>/deepseek-v4-flash/latest
```

Где `<folder-id>` — ID вашего каталога в Yandex Cloud.

### Reasoning (обязательно для DeepSeek)

DeepSeek — reasoning-модель. Если не включить параметры ниже, модель не будет
выдавать цепочку рассуждений.

```toml
model_supports_reasoning_summaries = true   # обязательно
model_reasoning_effort = "medium"           # "none" отключает reasoning!
model_reasoning_summary = "auto"
```


|Параметр|Обязательный|Описание|
|:---|:---|:---|
|`model_supports_reasoning_summaries`|**да**|Включает блок `reasoning` в запросе. Без него DeepSeek не будет отдавать reasoning|
|`model_reasoning_effort`|**да**|Уровень. Для DeepSeek: `"medium"` или `"high"`. **`"none"` отключает reasoning**|
|`model_reasoning_summary`|нет|`"auto"`|

### `developer_instructions` — отключение apply_patch

Некоторые провайдеры/окружения не поддерживают `apply_patch`. В таких случаях
инструкция запрещает Codex использовать этот инструмент:

```toml
developer_instructions = """
Do NOT use the apply_patch tool — it is not available in this environment.
Use shell commands (cat, sed, cp, tee, heredocs) or write files directly to make changes.
"""
```

### `model_catalog_json` — описание модели

Без model catalog Codex не знает параметры модели: контекстное окно,
поддержку инструментов, тип shell и т.д.

```json
{
  "models": [
    {
      "slug": "gpt://<folder-id>/deepseek-v4-flash/latest",
      "display_name": "DeepSeek V4 Flash",
      "shell_type": "default",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 30,
      "base_instructions": "You are Codex, a coding agent.",
      "supports_reasoning_summaries": true,
      "default_reasoning_summary": "auto",
      "support_verbosity": false,
      "supported_reasoning_levels": [],
      "supports_parallel_tool_calls": true,
      "context_window": 1000000,
      "max_context_window": 1000000,
      "effective_context_window_percent": 90,
      "input_modalities": ["text"],
      "supports_search_tool": false,
      "use_responses_lite": false,
      "tool_mode": null,
      "multi_agent_version": null,
      "truncation_policy": {"mode": "bytes", "limit": 10000}
    }
  ]
}
```

### `[model_providers.yc]` — настройки провайдера


|Параметр|Значение|Описание|
|:---|:---|:---|
|`name`|`"Yandex Cloud"`|Отображаемое имя|
|`base_url`|`"https://ai.api.cloud.yandex.net/v1"`|Базовый URL Responses API|
|`wire_api`|`"responses"`|DeepSeek работает через Responses API|
|`env_key`|`"YC_API_KEY"`|Переменная окружения с API-ключом|
|`requires_openai_auth`|`false`|YC не использует OpenAI-логин|
|`supports_websockets`|`false`|YC не поддерживает WebSocket|
|`env_http_headers`|`{ "x-folder-id" = "YC_FOLDER_ID", "OpenAI-Project" = "YC_FOLDER_ID" }`|Каталог YC из переменной окружения|

### Переменные окружения


|Переменная|Откуда взять|
|:---|:---|
|`YC_API_KEY`|Static API key из Yandex Cloud|
|`YC_FOLDER_ID`|ID каталога в Yandex Cloud|

### `[tui]` — настройки интерфейса

```toml
[tui]
status_line = ["model-name", "context-used"]
status_line_use_colors = true
```
