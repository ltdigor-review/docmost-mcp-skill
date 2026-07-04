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

Вставьте такой же блок в файл конфигурации Claude Desktop.

Если в файле уже есть `mcpServers`, добавьте внутрь него только блок `"docmost": { ... }`, а не второй `mcpServers`.

### Codex

В Codex этот JSON обычно не вставляют. Добавьте MCP server через настройки Codex или через CLI/конфиг, если вы настраиваете Codex локально.

Нужные значения:

```text
Name: docmost
URL: https://docmost.offercore.ru/mcp
Transport: Streamable HTTP
Header: Authorization: Bearer dcmcp_YOUR_TOKEN
```

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

## Как пользоваться

Спросите агента обычным текстом:

```text
Как подготовиться к трудоустройству?
```

Агент должен сам вызвать `search_pages`, затем `get_page`, и дать ответ со ссылками на статьи Docmost.

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
