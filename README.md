# Docmost MCP Skill

Skill для AI-агента, который читает Docmost как базу знаний через MCP.

Зачем это нужно: вы задаете агенту вопрос, агент сам ищет статьи в Docmost, читает нужные страницы и отвечает со ссылками на источники. Доступ только на чтение.

## Что понадобится

- аккаунт в Docmost;
- MCP-токен `dcmcp_...`;
- MCP-клиент или агент, куда можно добавить MCP server;
- этот репозиторий со skill.

## Шаг 1. Получите токен в Docmost

Откройте Docmost:

```text
Account -> API keys -> MCP tokens -> Create MCP token
```

Скопируйте токен сразу. Docmost показывает его один раз.

Не вставляйте настоящий токен в публичные файлы, GitHub, README или чат.

## Шаг 2. Скачайте skill

Вариант через Git:

```bash
git clone https://github.com/LTDigor/docmost-mcp-skill.git
```

Если Git не установлен, скачайте ZIP:

```text
https://github.com/LTDigor/docmost-mcp-skill/archive/refs/heads/main.zip
```

После распаковки вам нужен файл:

```text
skills/docmost-mcp/SKILL.md
```

## Шаг 3. Подключите MCP server

Этот JSON не нужно вставлять в терминал. Его вставляют в настройки MCP-клиента.

Важно: назовите MCP server именно `docmost`. Skill ниже ожидает такое имя сервера.

### Cursor

В проекте создайте файл:

```text
.cursor/mcp.json
```

Вставьте туда:

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

Замените `dcmcp_YOUR_TOKEN` на токен из Docmost. После этого перезапустите Cursor или обновите MCP servers.

### Claude Desktop

Откройте настройки Claude Desktop:

```text
Settings -> Developer -> Edit Config
```

В файл конфигурации вставьте блок `docmost` внутрь `mcpServers`:

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

Если в файле уже есть `mcpServers`, добавьте внутрь него только блок `"docmost": { ... }`, а не второй `mcpServers`.

### Codex

В Codex добавьте MCP server через настройки приложения или локальный конфиг MCP. Не вставляйте JSON в обычный чат с агентом.

Нужные значения:

```text
Name: docmost
URL: https://docmost.offercore.ru/mcp
Transport: Streamable HTTP
Header: Authorization: Bearer dcmcp_YOUR_TOKEN
```

Если Codex просит JSON-конфиг, используйте тот же блок, что в разделе Cursor.

### Другой MCP-клиент

Ищите настройку с названием `MCP servers`, `mcp.json`, `Connectors` или `Tools`.

Нужные значения те же:

```text
URL: https://docmost.offercore.ru/mcp
Transport: Streamable HTTP
Header: Authorization: Bearer dcmcp_YOUR_TOKEN
```

## Шаг 4. Подключите skill к агенту

Если агент просит путь к файлу skill, укажите:

```text
skills/docmost-mcp/SKILL.md
```

Если агент просит папку со skill, укажите:

```text
skills/docmost-mcp
```

После этого перезапустите агента или обновите MCP servers.

## Шаг 5. Проверьте подключение

После настройки откройте агента и попросите его проверить Docmost MCP:

```text
Проверь, что MCP server `docmost` доступен. Вызови `list_spaces` или найди страницу по запросу `трудоустройство`.
```

Успешный результат: агент видит инструменты Docmost MCP и может вызвать `list_spaces`, `search_pages` или `get_page`.

Если агент не видит инструменты:

- проверьте, что server называется `docmost`;
- проверьте URL `https://docmost.offercore.ru/mcp`;
- проверьте заголовок `Authorization: Bearer dcmcp_YOUR_TOKEN`;
- перезапустите MCP-клиент или обновите список MCP servers;
- если ошибка `401`, создайте новый MCP-токен в Docmost и замените старый.

## Как пользоваться

Спросите агента обычным текстом:

```text
Как подготовиться к трудоустройству?
```

Агент должен сам вызвать `search_pages`, затем `get_page`, и дать ответ со ссылками на статьи Docmost.

Пример хорошего ответа заканчивается так:

```text
Sources:
- https://docmost.offercore.ru/s/.../p/...
```

## Если хотите, чтобы агент настроил все сам

Отправьте агенту ссылку на репозиторий:

```text
https://github.com/LTDigor/docmost-mcp-skill
```

И дайте ему такой запрос:

```text
Скачай этот репозиторий, подключи skill `skills/docmost-mcp/SKILL.md`, добавь MCP server Docmost:

URL: https://docmost.offercore.ru/mcp
Transport: Streamable HTTP
Header: Authorization: Bearer dcmcp_MY_TOKEN

Замени `dcmcp_MY_TOKEN` на мой реальный токен. Не печатай токен в логах и не сохраняй его в публичных файлах.
```

## Важно

Docmost MCP только читает данные. Агент не сможет создавать, редактировать или удалять страницы.

Не публикуйте настоящий `dcmcp_...` токен в GitHub, README, скриншотах, логах или чатах. Если токен случайно попал в публичное место, удалите его в Docmost и создайте новый.
