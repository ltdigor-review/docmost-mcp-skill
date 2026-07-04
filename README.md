# Docmost MCP Skill

## Описание

Этот skill нужен AI-агенту, чтобы читать Docmost через MCP: искать страницы, открывать нужные материалы и отвечать со ссылками на источники.

Это инструкция только для OfferCore Docmost: `https://docmost.offercore.ru/mcp`.

Это не MCP-сервер и не плагин Docmost. Сам Docmost MCP уже должен быть доступен в OfferCore Docmost.

## Перед началом

- аккаунт в Docmost;
- MCP-токен вида `dcmcp_...`;
- Codex или другой агент с поддержкой MCP;
- ссылка на этот репозиторий:

```text
https://github.com/ltdigor-review/docmost-mcp-skill
```

URL MCP:

```text
https://docmost.offercore.ru/mcp
```

## Установка

### Вариант 1. Установка агентом

Рекомендуемый способ: отправьте агенту ссылку на репозиторий и попросите настроить все самому.

Скопируйте текст:

```text
Настрой Docmost MCP из репозитория:
https://github.com/ltdigor-review/docmost-mcp-skill

Параметры:
- server name: docmost
- URL: https://docmost.offercore.ru/mcp
- transport: streamable_http
- auth: Bearer token
- token: dcmcp_YOUR_TOKEN

Сделай:
1. Установи skill в ~/.codex/skills/docmost-mcp
2. Добавь или проверь ~/.codex/config.toml:

[mcp_servers.docmost]
url = "https://docmost.offercore.ru/mcp"
bearer_token_env_var = "DOCMOST_MCP_TOKEN"

3. Сохрани токен в DOCMOST_MCP_TOKEN для Codex, не печатай его в ответе.
4. Проверь подключение:
   - прямым MCP initialize запросом
   - командой codex mcp list
5. Если текущий чат не видит инструменты MCP, скажи перезапустить Codex.
```

### Вариант 2. Установка в Codex руками

Установите skill:

```bash
mkdir -p ~/.codex/skills
rm -rf /tmp/docmost-mcp-skill
git clone https://github.com/ltdigor-review/docmost-mcp-skill.git /tmp/docmost-mcp-skill
rm -rf ~/.codex/skills/docmost-mcp
cp -R /tmp/docmost-mcp-skill/skills/docmost-mcp ~/.codex/skills/docmost-mcp
```

Добавьте MCP server в настройках Codex:

```text
Name: docmost
URL: https://docmost.offercore.ru/mcp
Transport: Streamable HTTP
Header: Authorization: Bearer dcmcp_YOUR_TOKEN
```

Перезапустите Codex.

## Использование

Проверьте подключение:

```text
Проверь, что MCP server `docmost` доступен. Вызови `list_spaces` или найди страницу по запросу `трудоустройство`.
```

После проверки задайте вопрос по базе знаний:

```text
Как подготовиться к трудоустройству?
```

Агент найдет подходящие страницы в Docmost, прочитает их и даст ответ со ссылками на источники.
