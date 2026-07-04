---
name: docmost-mcp
description: Use Docmost as a read-only MCP knowledge base. Search pages, read relevant sources, answer from retrieved content, and cite sourceUrl links.
---

# Docmost MCP

Use this skill when a user asks a question that should be answered from Docmost knowledge base content, or when the user explicitly mentions Docmost MCP.

## Preconditions

Docmost MCP must already be configured in the local MCP client.

If setup is the current task, configure the MCP server first, then verify it before answering that setup is complete.

Expected server:

```text
docmost
```

Expected endpoint for OfferCore:

```text
https://docmost.offercore.ru/mcp
```

For another Docmost instance, use that instance's `/mcp` endpoint.

Expected transport:

```text
Streamable HTTP
```

Expected token format:

```text
dcmcp_...
```

Do not ask the user for the token unless setup or authentication is the current task. Prefer client settings, secret storage, or local MCP config over pasting tokens into chat. Never print a full token in chat, logs, docs, diffs, or examples.

If the server is configured under a different name, either use that available server name if the client exposes it clearly, or tell the user to rename it to `docmost` for this skill.

## Tool scope

Docmost MCP is read-only.

Allowed tools:

- `search_pages`
- `get_page`
- `list_spaces`
- `list_child_pages`
- `get_mcp_context`

Do not expect write, delete, admin, database, cookie, or browser-session access through MCP.

If the user asks to create, edit, delete, publish, invite users, change permissions, or perform admin work, explain that this MCP server cannot do that and ask for an appropriate write/admin workflow instead.

Do not fall back to browser sessions, cookies, direct database access, write APIs, admin panels, or local exports unless the user explicitly asks for that separate workflow.

## Default workflow

1. Search with `search_pages` using the user's question.
2. If the first search is weak, rewrite the query with domain terms and search again.
3. Select 2 to 5 relevant pages.
4. Read selected pages with `get_page`.
5. Answer only from retrieved content unless clearly labeled as general knowledge.
6. Cite sources with `sourceUrl` links at the end.

If the answer needs one exact article and search returns a clear match, reading one page is enough.

For setup verification, use `list_spaces` or a small `search_pages` query such as `трудоустройство`. Do not read unrelated pages just to prove the connection works.

## Source handling

Prefer direct page evidence over broad summaries.

When sources disagree, say so and cite both pages.

If Docmost does not contain enough information, say what is missing. Do not invent missing policy, process, pricing, student guidance, internal decisions, or operational details.

## Search tips

Use the user's words first.

If needed, add synonyms, product names, team names, or Russian and English variants.

For broad questions, call `list_spaces` before searching and restrict search to the most relevant space if the MCP tool supports it.

## Error handling

### 401 Unauthorized

Tell the user the MCP token is missing, invalid, expired, revoked, or copied incorrectly. Ask them to create a new `dcmcp_...` token if needed.

### 403 Forbidden

Tell the user MCP may be disabled, the user may not have access, or the token space scope may exclude the page.

### 429 Too Many Requests

Slow down. Retry only if the client workflow allows it. If rate limiting continues, tell the user the server limit needs adjustment.

### No relevant pages

Try one narrower search and one broader search. If both fail, answer that Docmost did not return a relevant source and state the searches you tried.

### MCP tools not visible

Tell the user the MCP client has not loaded the `docmost` server. Ask them to check the server name, endpoint, transport, authorization header, and then restart or refresh the MCP client.

## Output format

Keep answers concise.

End with sources when Docmost content was used:

```text
Sources:
- https://docmost.offercore.ru/s/.../p/...
- https://docmost.offercore.ru/s/.../p/...
```

Do not cite pages that were not read with `get_page`.
