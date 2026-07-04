# Docmost MCP Skill

Skill для AI-агента, который читает Docmost как базу знаний через MCP.

Зачем это нужно: агент сам находит статьи в Docmost, читает их, отвечает пользователю и прикладывает ссылки на источники. Доступ только на чтение.

## Скачать

Через Git:

```bash
git clone https://github.com/LTDigor/docmost-mcp-skill.git
```

Или скачайте архив:

```text
https://github.com/LTDigor/docmost-mcp-skill/archive/refs/heads/main.zip
```

## Установка

1. Создайте MCP-токен в Docmost:

```text
Account -> API keys -> MCP tokens -> Create MCP token
```

2. Добавьте MCP server в клиент:

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

Transport: `Streamable HTTP`.

3. Подключите skill к агенту:

```text
skills/docmost-mcp/SKILL.md
```

или папку:

```text
skills/docmost-mcp
```

## Использование

Спросите агента что-то по базе знаний, например:

```text
Как подготовиться к трудоустройству?
```

Агент должен использовать `search_pages`, затем `get_page`, и ответить со ссылками на источники.

