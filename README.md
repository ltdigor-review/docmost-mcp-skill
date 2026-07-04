# Docmost MCP Skill

Готовый skill для AI-агента, который умеет читать Docmost как базу знаний через MCP.

Важно: это не MCP-сервер и не плагин Docmost. Это инструкция для агента. Сам Docmost MCP должен уже работать на стороне вашего Docmost.

Что получится после настройки:

- агент увидит MCP server с именем `docmost`;
- агент сможет искать страницы Docmost через `search_pages`;
- агент сможет читать найденные страницы через `get_page`;
- агент будет отвечать по материалам Docmost и давать ссылки на источники;
- агент не сможет создавать, редактировать или удалять страницы, потому что этот MCP доступ только для чтения.

## Что понадобится

- аккаунт в Docmost;
- MCP-токен вида `dcmcp_...`;
- MCP-клиент: Codex, Cursor, Claude Desktop или другой клиент с поддержкой MCP;
- этот репозиторий.

## Короткая схема

1. Создать MCP token в Docmost.
2. Добавить MCP server `docmost` в свой MCP-клиент.
3. Установить skill `docmost-mcp`.
4. Перезапустить клиента.
5. Проверить, что агент видит tools `list_spaces`, `search_pages`, `get_page`.

Если вы не понимаете, куда вставлять JSON, не вставляйте его в терминал. JSON из этой инструкции вставляется в настройки MCP-клиента.

## Шаг 1. Получите MCP token в Docmost

Откройте Docmost и создайте MCP token:

```text
Account -> API keys -> MCP tokens -> Create MCP token
```

Скопируйте токен сразу. Обычно Docmost показывает полный токен только один раз.

Токен должен выглядеть примерно так:

```text
dcmcp_XXXXXXXXXXXXXXXX
```

Никогда не публикуйте настоящий токен:

- в GitHub;
- в README;
- в скриншотах;
- в логах;
- в публичных чатах;
- в issue или pull request.

Если токен случайно попал в публичное место, удалите его в Docmost и создайте новый.

## Шаг 2. Выберите URL Docmost MCP

Для OfferCore:

```text
https://docmost.offercore.ru/mcp
```

Для другого Docmost замените домен:

```text
https://YOUR_DOCMOST_DOMAIN/mcp
```

Дальше в примерах используется OfferCore URL. Если у вас другой Docmost, замените URL везде.

## Шаг 3. Подключите MCP server

Имя server должно быть именно:

```text
docmost
```

Skill ожидает это имя. Если назвать server иначе, агент может не понять, что надо использовать именно Docmost MCP.

### Вариант A. Cursor

В проекте создайте файл:

```text
.cursor/mcp.json
```

Вставьте:

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

Что заменить:

- `https://docmost.offercore.ru/mcp` - на ваш URL, если у вас другой Docmost;
- `dcmcp_YOUR_TOKEN` - на реальный MCP token.

После этого перезапустите Cursor или обновите MCP servers.

### Вариант B. Claude Desktop

Откройте:

```text
Settings -> Developer -> Edit Config
```

В конфиг добавьте server `docmost` внутрь `mcpServers`:

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

Если в файле уже есть `mcpServers`, не создавайте второй `mcpServers`. Добавьте только этот блок:

```json
"docmost": {
  "type": "http",
  "url": "https://docmost.offercore.ru/mcp",
  "headers": {
    "Authorization": "Bearer dcmcp_YOUR_TOKEN"
  }
}
```

Следите за запятыми в JSON. Если рядом уже есть другой server, между блоками нужна запятая.

После изменения конфига перезапустите Claude Desktop.

### Вариант C. Codex

#### 1. Установите skill

Самый простой способ через Git:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/ltdigor-review/docmost-mcp-skill.git /tmp/docmost-mcp-skill
cp -R /tmp/docmost-mcp-skill/skills/docmost-mcp ~/.codex/skills/docmost-mcp
```

Если skill уже был установлен и вы хотите заменить его свежей версией:

```bash
rm -rf ~/.codex/skills/docmost-mcp
cp -R /tmp/docmost-mcp-skill/skills/docmost-mcp ~/.codex/skills/docmost-mcp
```

После установки перезапустите Codex.

#### 2. Добавьте MCP server

В настройках Codex добавьте MCP server со значениями:

```text
Name: docmost
URL: https://docmost.offercore.ru/mcp
Transport: Streamable HTTP
Header: Authorization: Bearer dcmcp_YOUR_TOKEN
```

Если Codex просит JSON, используйте:

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

Не вставляйте реальный токен в обычный чат с агентом, если можно добавить его через настройки приложения, secret storage или локальный конфиг.

### Вариант D. Другой MCP-клиент

Ищите настройку с похожим названием:

- `MCP servers`;
- `mcp.json`;
- `Connectors`;
- `Tools`;
- `Developer settings`.

Нужные значения:

```text
Server name: docmost
URL: https://docmost.offercore.ru/mcp
Transport: Streamable HTTP
Authorization header: Bearer dcmcp_YOUR_TOKEN
```

Если клиент не принимает `type: "http"`, найдите в его документации вариант для Streamable HTTP MCP server.

## Шаг 4. Подключите skill к агенту

Если агент просит путь к файлу:

```text
skills/docmost-mcp/SKILL.md
```

Если агент просит путь к папке:

```text
skills/docmost-mcp
```

Для Codex после копирования в `~/.codex/skills/docmost-mcp` skill должен появиться после перезапуска.

## Шаг 5. Проверьте подключение

Откройте агента и попросите:

```text
Проверь, что MCP server `docmost` доступен. Вызови `list_spaces` или найди страницу по запросу `трудоустройство`.
```

Успешный результат:

- агент видит tools Docmost MCP;
- агент может вызвать `list_spaces`;
- агент может вызвать `search_pages`;
- агент может вызвать `get_page`.

Пример нормальной проверки:

```text
MCP server `docmost` доступен. `list_spaces` вернул список пространств. Поиск по `трудоустройство` возвращает страницы Docmost.
```

## Как пользоваться

Задайте агенту обычный вопрос:

```text
Как подготовиться к трудоустройству?
```

Агент должен:

1. вызвать `search_pages`;
2. выбрать подходящие страницы;
3. вызвать `get_page`;
4. ответить по содержимому страниц;
5. дать ссылки на источники.

Хороший ответ заканчивается источниками:

```text
Sources:
- https://docmost.offercore.ru/s/.../p/...
```

## Если хотите, чтобы агент настроил все сам

Так можно делать только если вы доверяете локальной агент-сессии.

Лучше безопасный вариант: сначала сами добавьте токен через настройки MCP-клиента или secret storage, а агенту дайте только ссылку на репозиторий и URL.

Запрос агенту:

```text
Скачай репозиторий https://github.com/ltdigor-review/docmost-mcp-skill.
Установи skill из `skills/docmost-mcp/SKILL.md`.
Подключи MCP server:

Name: docmost
URL: https://docmost.offercore.ru/mcp
Transport: Streamable HTTP
Header: Authorization: Bearer <мой MCP token>

Не печатай полный token в логах, ответах, README, git diff или публичных файлах.
Если можно, используй настройки клиента или secret storage вместо вставки token в файлы проекта.
```

## Частые проблемы

### Агент не видит Docmost tools

Проверьте:

- server называется `docmost`;
- URL заканчивается на `/mcp`;
- выбран Streamable HTTP transport;
- header называется `Authorization`;
- значение header начинается с `Bearer `;
- после `Bearer ` стоит реальный `dcmcp_...` token;
- MCP-клиент был перезапущен после изменения конфига.

### Ошибка 401 Unauthorized

Обычно значит одно из этого:

- token не вставлен;
- token скопирован не полностью;
- token удален или истек;
- вместо MCP token вставлен другой API key;
- забыли префикс `Bearer `.

Решение: создайте новый MCP token в Docmost и замените старый.

### Ошибка 403 Forbidden

Возможные причины:

- MCP выключен на стороне Docmost;
- у пользователя нет доступа к нужному пространству;
- token ограничен и не видит нужные страницы.

### Ошибка 404 Not Found

Проверьте URL. Обычно правильный URL выглядит так:

```text
https://YOUR_DOCMOST_DOMAIN/mcp
```

Не добавляйте лишние пути, если ваша установка Docmost не требует этого.

### Ошибка JSON config

Чаще всего проблема в запятых или в том, что создан второй `mcpServers`.

Правильно:

```json
{
  "mcpServers": {
    "first-server": {
      "type": "http",
      "url": "https://example.com/mcp"
    },
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

Неправильно:

```json
{
  "mcpServers": {
    "first-server": {}
  },
  "mcpServers": {
    "docmost": {}
  }
}
```

## Проверено

Этот репозиторий описывает стандартную настройку Streamable HTTP MCP server.

Статус клиентов:

- Codex: инструкция установки skill добавлена;
- Cursor: JSON пример добавлен;
- Claude Desktop: JSON пример добавлен;
- другие MCP-клиенты: используйте URL, transport и Authorization header из инструкции.

Если конкретный клиент требует другой формат конфига, используйте его документацию, но сохраняйте те же значения: server name `docmost`, URL `/mcp`, header `Authorization: Bearer ...`.

## Что этот skill не делает

Он не дает агенту права:

- создавать страницы;
- редактировать страницы;
- удалять страницы;
- приглашать пользователей;
- менять права;
- читать базу данных напрямую;
- заходить в браузерную сессию Docmost;
- использовать cookies;
- обходить ограничения доступа.

Если нужна запись или админка, нужен отдельный write/admin workflow. Этот репозиторий только для read-only поиска и чтения через MCP.

