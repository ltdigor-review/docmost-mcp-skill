# Docmost MCP Skill

## Описание

Этот skill помогает AI-агенту читать Docmost через MCP: искать страницы, открывать нужные материалы и отвечать со ссылками на источники.

Это не MCP-сервер и не плагин Docmost. Сам Docmost MCP уже должен быть доступен в вашем Docmost.

## Перед началом

- аккаунт в Docmost;
- MCP token вида `dcmcp_...`;
- Codex или другой агент с поддержкой MCP;
- ссылка на этот репозиторий:

```text
https://github.com/ltdigor-review/docmost-mcp-skill
```

Для OfferCore используйте URL:

```text
https://docmost.offercore.ru/mcp
```

Если у вас другой Docmost, замените URL на свой:

```text
https://YOUR_DOCMOST_DOMAIN/mcp
```

## Установка

### Вариант 1. Установка агентом

Это основной вариант. Отправьте агенту ссылку на репозиторий и попросите настроить все самому.

Скопируйте текст:

```text
Настрой мне Docmost MCP по инструкции из репозитория:
https://github.com/ltdigor-review/docmost-mcp-skill

Сделай так:
1. Скачай репозиторий.
2. Установи skill `skills/docmost-mcp`.
3. Добавь MCP server с именем `docmost`.
4. Используй URL `https://docmost.offercore.ru/mcp`.
5. Используй transport `Streamable HTTP`.
6. Добавь header `Authorization: Bearer <мой MCP token>`.
7. Проверь подключение через `list_spaces` или `search_pages`.
8. Напиши, что изменил и где.

Не печатай полный token в ответах, логах, git diff или публичных файлах.
Если нужен пароль, OAuth, MFA, CAPTCHA или системное разрешение, остановись и попроси меня выполнить этот шаг.
Если не можешь изменить настройки MCP сам, дай точную ручную инструкцию.
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

После проверки задавайте вопросы по базе знаний:

```text
Как подготовиться к трудоустройству?
```

Агент найдет подходящие страницы в Docmost, прочитает их и ответит со ссылками на источники.
