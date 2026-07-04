# Docmost MCP: подключение AI-агента к базе знаний

Этот репозиторий описывает, как подключить внешний MCP-compatible AI-агент к Docmost как к read-only источнику знаний.

MCP-доступ нужен для сценария: пользователь задает вопрос агенту, агент ищет релевантные статьи в Docmost, читает их как чистый Markdown/текст, возвращает ответ и прикладывает ссылки на источники.

## Что умеет Docmost MCP

Docmost MCP дает агенту только чтение:

- искать страницы: `search_pages`
- открыть страницу: `get_page`
- посмотреть доступные пространства: `list_spaces`
- посмотреть дочерние страницы: `list_child_pages`
- проверить текущий workspace/user/token scope: `get_mcp_context`

Записи, удаления, изменения страниц и админские операции через MCP недоступны.

## Безопасность

Docmost MCP не использует cookies, browser session, admin session и обычные API keys.

Для подключения нужен отдельный MCP-токен формата:

```text
dcmcp_...
```

Агент видит только то, что видит пользователь, который создал токен. Если у токена задан scope по пространствам, доступ дополнительно ограничивается этими пространствами.

MCP не отдает:

- raw ProseMirror JSON
- raw DB fields
- cookies
- внутренние секреты
- storage paths
- internal config
- write tools

## Как администратору включить MCP

На сервере должен быть включен глобальный флаг:

```env
MCP_SERVER_ENABLED=true
```

Дополнительные настройки:

```env
MCP_RATE_LIMIT_PER_MINUTE=60
MCP_MAX_SEARCH_RESULTS=10
MCP_MAX_PAGE_CHARS=80000
```

После этого администратор включает MCP в интерфейсе Docmost:

```text
Settings -> AI settings -> MCP -> Enable
```

Если `MCP_SERVER_ENABLED=false`, workspace-тумблер в интерфейсе не даст создать рабочие MCP-токены.

## Как пользователю получить MCP-токен

1. Откройте Docmost.
2. Перейдите в `Account -> API keys`.
3. Найдите блок `MCP tokens`.
4. Нажмите `Create MCP token`.
5. Укажите имя токена, например `Cursor`, `Claude`, `Job agent`.
6. При необходимости ограничьте токен конкретными пространствами.
7. Выберите срок действия.
8. Скопируйте токен сразу после создания.

Токен показывается только один раз. Если вы его потеряли, создайте новый и отзовите старый.

## Параметры подключения

Endpoint:

```text
https://YOUR_DOCMOST_DOMAIN/mcp
```

Transport:

```text
Streamable HTTP
```

Header:

```text
Authorization: Bearer dcmcp_...
```

Для OfferCore Docmost endpoint:

```text
https://docmost.offercore.ru/mcp
```

## Пример generic MCP client config

Формат у разных клиентов отличается, но смысл один: указать URL MCP endpoint и bearer token.

```json
{
  "mcpServers": {
    "docmost": {
      "type": "http",
      "url": "https://docmost.offercore.ru/mcp",
      "headers": {
        "Authorization": "Bearer dcmcp_YOUR_TOKEN"
      }
    }
  }
}
```

Если клиент отдельно спрашивает transport, выбирайте:

```text
Streamable HTTP
```

## Как агент должен использовать Docmost

Рекомендуемый порядок:

1. Вызвать `search_pages` с вопросом пользователя.
2. Выбрать 2-5 наиболее релевантных страниц.
3. Открыть их через `get_page`.
4. Ответить пользователю на своей стороне.
5. В конце ответа дать ссылки на источники из `sourceUrl`.

Пример поведения:

```text
Пользователь: Как подготовиться к трудоустройству?

Агент:
1. search_pages("подготовка к трудоустройству собеседование резюме")
2. get_page("...")
3. get_page("...")
4. Формирует ответ.
5. Добавляет источники:
   - https://docmost.offercore.ru/s/hr/p/...
```

## Что возвращает `get_page`

Страница возвращается в читаемом виде для модели:

- заголовки сохраняются как Markdown headings
- списки остаются списками
- таблицы остаются читаемыми таблицами
- ссылки остаются ссылками
- служебный ProseMirror JSON не возвращается

Пример:

```markdown
# Подготовка к собеседованию

## Перед интервью

- обновите резюме
- подготовьте ответы на частые вопросы
- проверьте ссылку на портфолио

Source: https://docmost.offercore.ru/s/hr/p/example
```

## Частые ошибки

### 401 Unauthorized

Причины:

- токен не передан
- передан обычный API key вместо MCP-токена
- токен скопирован не полностью
- токен отозван или истек

### 403 Forbidden

Причины:

- MCP выключен на сервере
- MCP выключен в workspace
- пользователь деактивирован
- страница недоступна пользователю
- страница вне space scope токена

### 429 Too Many Requests

Агент превысил rate limit. Уменьшите частоту запросов или настройте лимит на сервере.

### Агент не находит статью

Попробуйте:

- задать более конкретный запрос
- сначала вызвать `list_spaces`
- ограничить поиск нужным `spaceId`
- проверить, что пользователь сам видит эту статью в Docmost

## Как отозвать токен

Пользователь может отозвать свой токен:

```text
Account -> API keys -> MCP tokens -> Revoke
```

Администратор может управлять MCP-доступом в workspace settings.

## Минимальный чеклист проверки

- MCP включен через `MCP_SERVER_ENABLED=true`.
- MCP включен в workspace UI.
- Пользователь создал `dcmcp_...` токен.
- Клиент подключается к `https://YOUR_DOCMOST_DOMAIN/mcp`.
- Header содержит `Authorization: Bearer <token>`.
- `list_tools` показывает только read-only tools.
- `search_pages` возвращает релевантные страницы.
- `get_page` возвращает чистый Markdown и `sourceUrl`.

