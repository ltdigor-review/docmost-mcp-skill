# Docmost MCP skill

Русская версия ниже. English version follows.

## Русская версия

### Что это

Этот репозиторий содержит agent skill для работы с Docmost через MCP и короткую инструкцию по подключению.

Это не реализация Docmost MCP server. MCP endpoint уже должен существовать на стороне Docmost.

Docmost в этом сценарии работает как база знаний только для чтения. Агент может искать статьи, открывать нужные страницы, отвечать пользователю и прикладывать ссылки на источники.

Репозиторий состоит из двух частей:

- этот `README.md` с настройкой сервера, токена и MCP-клиента
- `skills/docmost-mcp/SKILL.md` с инструкцией для агента

README нужен человеку, который настраивает доступ. Skill нужен агенту, чтобы он правильно пользовался Docmost после подключения.

### Что умеет Docmost MCP

Доступны только read-only tools:

- `search_pages`: поиск страниц
- `get_page`: чтение страницы
- `list_spaces`: список доступных пространств
- `list_child_pages`: дочерние страницы
- `get_mcp_context`: текущий workspace, user и token scope

Через MCP нельзя создавать, редактировать или удалять страницы. Админские действия тоже недоступны.

### Безопасность

Docmost MCP не использует cookies, browser session, admin session или обычные API keys.

Для подключения нужен отдельный MCP-токен:

```text
dcmcp_...
```

Агент видит только то, что видит пользователь, создавший токен. Если токен ограничен конкретными пространствами, агент видит только эти пространства.

MCP не возвращает:

- raw ProseMirror JSON
- raw DB fields
- cookies
- внутренние секреты
- storage paths
- internal config
- write tools

### Включить MCP на сервере

На сервере Docmost включите глобальный флаг:

```env
MCP_SERVER_ENABLED=true
```

Дополнительные лимиты:

```env
MCP_RATE_LIMIT_PER_MINUTE=60
MCP_MAX_SEARCH_RESULTS=10
MCP_MAX_PAGE_CHARS=80000
```

Затем включите MCP в workspace:

```text
Settings -> AI settings -> MCP -> Enable
```

Если `MCP_SERVER_ENABLED=false`, workspace-тумблер не создаст рабочие MCP-токены.

### Создать MCP-токен

В Docmost:

1. Откройте `Account -> API keys`.
2. Найдите `MCP tokens`.
3. Нажмите `Create MCP token`.
4. Укажите имя, например `Cursor`, `Claude`, `Codex` или `Job agent`.
5. При необходимости ограничьте токен конкретными пространствами.
6. Выберите срок действия.
7. Скопируйте токен сразу после создания.

Токен показывается один раз. Если вы его потеряли, создайте новый и отзовите старый.

### Подключить MCP-клиент

Endpoint:

```text
https://YOUR_DOCMOST_DOMAIN/mcp
```

Endpoint для OfferCore Docmost:

```text
https://docmost.offercore.ru/mcp
```

Transport:

```text
Streamable HTTP
```

Header:

```text
Authorization: Bearer dcmcp_YOUR_TOKEN
```

Пример generic MCP config:

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

Если клиент отдельно спрашивает transport, выбирайте `Streamable HTTP`.

### Cursor

Добавьте новый MCP server в настройках Cursor:

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

После сохранения перезапустите Cursor или перезагрузите MCP servers.

### Claude Desktop

Откройте MCP config Claude Desktop и добавьте:

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

После сохранения перезапустите Claude Desktop.

### Codex

Добавьте Docmost MCP server в MCP config Codex:

```text
Name: docmost
Type: http
URL: https://docmost.offercore.ru/mcp
Transport: Streamable HTTP
Header: Authorization: Bearer dcmcp_YOUR_TOKEN
```

Затем установите или скопируйте skill:

```text
skills/docmost-mcp/SKILL.md
```

MCP config дает агенту tools. Skill объясняет, когда искать, какие страницы читать и как цитировать источники.

### Как агент должен работать

Рекомендуемый порядок:

1. Вызвать `search_pages` по вопросу пользователя.
2. Выбрать 2-5 подходящих страниц.
3. Открыть их через `get_page`.
4. Ответить только по найденному материалу.
5. В конце добавить ссылки из `sourceUrl`.

Пример:

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

### Что возвращает `get_page`

`get_page` возвращает страницу в виде, удобном для модели:

- заголовки остаются Markdown headings
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

### Проверка

- На сервере включен `MCP_SERVER_ENABLED=true`.
- MCP включен в workspace UI.
- Пользователь создал `dcmcp_...` токен.
- Клиент подключается к `https://YOUR_DOCMOST_DOMAIN/mcp`.
- Header содержит `Authorization: Bearer <token>`.
- `list_tools` показывает только read-only tools.
- `search_pages` возвращает релевантные страницы.
- `get_page` возвращает Markdown и `sourceUrl`.

### Частые ошибки

#### 401 Unauthorized

Проверьте:

- токен не передан
- передан обычный API key вместо MCP-токена
- токен скопирован не полностью
- токен отозван или истек

#### 403 Forbidden

Проверьте:

- MCP выключен на сервере
- MCP выключен в workspace
- пользователь деактивирован
- страница недоступна пользователю
- страница вне space scope токена

#### 429 Too Many Requests

Агент превысил rate limit. Уменьшите частоту запросов или настройте лимит на сервере.

#### Агент не находит статью

Попробуйте:

- задать более конкретный запрос
- сначала вызвать `list_spaces`
- ограничить поиск нужным `spaceId`
- проверить, что пользователь сам видит эту статью в Docmost

### Отозвать токен

Пользователь может отозвать свой токен:

```text
Account -> API keys -> MCP tokens -> Revoke
```

Администратор управляет MCP-доступом в workspace settings.

## English version

### What this is

This repository provides an agent skill for using Docmost through MCP, plus a short setup guide.

It does not implement a Docmost MCP server. The MCP endpoint must already exist on the Docmost side.

In this setup, Docmost is a read-only knowledge base. The agent can search pages, open the relevant ones, answer the user, and cite the Docmost sources it used.

The repository has two parts:

- this `README.md`, with server, token, and MCP client setup
- `skills/docmost-mcp/SKILL.md`, with agent instructions

The README is for the person setting up access. The skill is for the agent that will use Docmost after MCP is connected.

### What Docmost MCP can do

Docmost MCP exposes read-only tools:

- `search_pages`: search pages
- `get_page`: read a page
- `list_spaces`: list accessible spaces
- `list_child_pages`: list child pages
- `get_mcp_context`: inspect workspace, user, and token scope

MCP cannot create, edit, or delete pages. It cannot perform admin actions.

### Security

Docmost MCP does not use cookies, browser sessions, admin sessions, or regular API keys.

The user connects with a dedicated MCP token:

```text
dcmcp_...
```

The agent gets the same access as the user who created the token. If the token is scoped to specific spaces, the agent can only see those spaces.

MCP does not return:

- raw ProseMirror JSON
- raw database fields
- cookies
- internal secrets
- storage paths
- internal config
- write tools

### Enable MCP on the server

Enable the global flag on the Docmost server:

```env
MCP_SERVER_ENABLED=true
```

Optional limits:

```env
MCP_RATE_LIMIT_PER_MINUTE=60
MCP_MAX_SEARCH_RESULTS=10
MCP_MAX_PAGE_CHARS=80000
```

Then enable MCP in the workspace:

```text
Settings -> AI settings -> MCP -> Enable
```

If `MCP_SERVER_ENABLED=false`, the workspace toggle will not create working MCP tokens.

### Create an MCP token

In Docmost:

1. Open `Account -> API keys`.
2. Find `MCP tokens`.
3. Click `Create MCP token`.
4. Enter a name, such as `Cursor`, `Claude`, `Codex`, or `Job agent`.
5. Scope the token to specific spaces if needed.
6. Choose the expiration period.
7. Copy the token right after creating it.

The token is shown once. If you lose it, create a new one and revoke the old one.

### Connect an MCP client

Endpoint:

```text
https://YOUR_DOCMOST_DOMAIN/mcp
```

OfferCore Docmost endpoint:

```text
https://docmost.offercore.ru/mcp
```

Transport:

```text
Streamable HTTP
```

Header:

```text
Authorization: Bearer dcmcp_YOUR_TOKEN
```

Example generic MCP config:

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

If the client asks for transport separately, choose `Streamable HTTP`.

### Cursor

Add a new MCP server in Cursor settings:

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

After saving, restart Cursor or reload MCP servers.

### Claude Desktop

Open the Claude Desktop MCP config and add:

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

Restart Claude Desktop after saving.

### Codex

Add the Docmost MCP server to the Codex MCP config:

```text
Name: docmost
Type: http
URL: https://docmost.offercore.ru/mcp
Transport: Streamable HTTP
Header: Authorization: Bearer dcmcp_YOUR_TOKEN
```

Then install or copy the skill:

```text
skills/docmost-mcp/SKILL.md
```

The MCP config gives the agent tools. The skill tells it when to search, which pages to read, and how to cite sources.

### How the agent should work

Recommended flow:

1. Call `search_pages` with the user's question.
2. Pick 2 to 5 relevant pages.
3. Open them with `get_page`.
4. Answer only from the retrieved material.
5. Add `sourceUrl` links at the end.

Example:

```text
User: How should I prepare for employment?

Agent:
1. search_pages("employment preparation interview resume")
2. get_page("...")
3. get_page("...")
4. Writes the answer.
5. Adds sources:
   - https://docmost.offercore.ru/s/hr/p/...
```

### What `get_page` returns

`get_page` returns model-friendly content:

- headings stay as Markdown headings
- lists stay as lists
- tables stay readable
- links stay as links
- service ProseMirror JSON is not returned

Example:

```markdown
# Interview preparation

## Before the interview

- update your resume
- prepare answers to common questions
- check your portfolio link

Source: https://docmost.offercore.ru/s/hr/p/example
```

### Validation

- The server has `MCP_SERVER_ENABLED=true`.
- MCP is enabled in the workspace UI.
- The user created a `dcmcp_...` token.
- The client connects to `https://YOUR_DOCMOST_DOMAIN/mcp`.
- Header contains `Authorization: Bearer <token>`.
- `list_tools` shows only read-only tools.
- `search_pages` returns relevant pages.
- `get_page` returns Markdown and `sourceUrl`.

### Common errors

#### 401 Unauthorized

Check whether:

- the token was not sent
- a regular API key was used instead of an MCP token
- the token was copied incompletely
- the token was revoked or expired

#### 403 Forbidden

Check whether:

- MCP is disabled on the server
- MCP is disabled in the workspace
- the user is deactivated
- the page is unavailable to the user
- the page is outside the token space scope

#### 429 Too Many Requests

The agent exceeded the rate limit. Reduce request frequency or change the server limit.

#### Agent cannot find a page

Try this:

- use a more specific query
- call `list_spaces` first
- limit search to the relevant `spaceId`
- confirm that the token owner can see the page in Docmost

### Revoke a token

A user can revoke their own token:

```text
Account -> API keys -> MCP tokens -> Revoke
```

Admins can manage MCP access in workspace settings.
