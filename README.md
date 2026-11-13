# Rolev.ai Job Search: MCP server

Search current US job openings, read from employers' own career sites, from Claude, Claude Code, Cursor, VS Code
or any MCP client that supports OAuth.

| | |
|---|---|
| Endpoint | `https://rolev.ai/mcp` (Streamable HTTP) |
| Sign-in | OAuth 2.1 with your Rolev.ai account (a free account is enough) |
| Access | Read-only: searches and reads public job postings |
| Registry name | `ai.rolev/job-search` |

This repository holds the documentation and the [MCP Registry](https://registry.modelcontextprotocol.io) manifest
([`server.json`](server.json)). Rolev.ai hosts the server itself, so there is nothing to install or run.

## Tools

| Tool | What it does | Inputs |
|---|---|---|
| `search_jobs` | Finds current US openings by role, with an optional place, remote, or employer. Returns up to 10 per page, each with a Rolev.ai link, plus the total. | `role`, `place`, `remote`, `company`, `page` |
| `get_job` | One job's posting text (up to 2,500 characters), pay, place and type, with its Rolev.ai page and the employer's own posting. | `id` (from `search_jobs`) |
| `list_roles` | The roles Rolev.ai covers, by sector, with how many are open. Useful when a search finds no matching role. | `sector` (optional) |

All three tools are annotated `readOnlyHint: true` and `openWorldHint: true`, because their results are third-party
job postings. Free text is matched to Rolev.ai's role catalog, so ask with a plain job title ("registered nurse",
"software developer"). A place can be a city and state ("Austin, TX"), a state ("Texas") or "remote".

## Connect

### Claude (desktop and web)

1. Open **Settings → Connectors** and choose **Add custom connector**.
2. Name it **Rolev.ai** and paste `https://rolev.ai/mcp`. Leave the client ID and secret empty.
3. Claude opens Rolev.ai. Sign in if asked, then press **Allow**.
4. Start a chat, enable the connector and ask for jobs.

### Claude Code

```bash
claude mcp add --transport http rolev https://rolev.ai/mcp
```

Then run `/mcp` inside Claude Code to sign in to Rolev.ai in your browser.

### Cursor

Add to `~/.cursor/mcp.json`, or to `.cursor/mcp.json` in a project ([example](examples/cursor.mcp.json)):

```json
{
  "mcpServers": {
    "rolev": { "url": "https://rolev.ai/mcp" }
  }
}
```

### VS Code

Add to `.vscode/mcp.json` ([example](examples/vscode.mcp.json)):

```json
{
  "servers": {
    "rolev": { "type": "http", "url": "https://rolev.ai/mcp" }
  }
}
```

### Other clients

Any client that supports Streamable HTTP and the MCP authorization spec (OAuth 2.1 with dynamic client registration)
can connect to `https://rolev.ai/mcp`. The client finds the sign-in on its own from the server's 401 response.
ChatGPT is not supported yet.

### Things to ask

- "Find remote registered nurse jobs."
- "Software developer jobs in Austin, TX. Show the pay where it's listed."
- "Which healthcare roles does Rolev.ai cover?"
- "Show me the details of the second job."

## Sign-in and permissions

- A call without a token gets `401` with a `WWW-Authenticate` header that points at
  [`/.well-known/oauth-protected-resource/mcp`](https://rolev.ai/.well-known/oauth-protected-resource/mcp).
- Rolev.ai is its own authorization server
  ([metadata](https://rolev.ai/.well-known/oauth-authorization-server)). It uses dynamic client registration
  (RFC 7591) and the authorization code flow with PKCE (S256), accepts public clients only, and has one scope:
  `jobs:read`.
- Access tokens last 24 hours. Refresh tokens last 90 days and rotate on every use.
- Your connected assistants are listed on Rolev.ai under **Profile → Answers Vault → Connected assistants**.
  Disconnecting one ends its access at once.

## Privacy and safety

- **Read-only.** The tools search and read public job postings only.
- **No access to your resume or profile.** The server never applies to a job for you. On Rolev.ai you review an
  application and press Submit yourself.
- Links in answers point to Rolev.ai pages and carry `utm_source=mcp`, so Rolev.ai can count visits that come from
  assistants.
- [Privacy policy](https://rolev.ai/privacy) · [Terms](https://rolev.ai/terms)

## Limits

- 30 requests a minute per account. The limit is shared with Rolev.ai's A2A agent and its in-browser WebMCP tools,
  and past it you get `429`.
- US job postings only.
- A tool error comes back as a tool result with one plain sentence (`isError: true`).

## Discovery

| Document | URL |
|---|---|
| MCP Server Card | https://rolev.ai/.well-known/mcp/server-card.json |
| Protected resource metadata (RFC 9728) | https://rolev.ai/.well-known/oauth-protected-resource/mcp |
| Authorization server metadata (RFC 8414) | https://rolev.ai/.well-known/oauth-authorization-server |
| Page for people | https://rolev.ai/ai |

## Support

Open an issue in this repository.

## License

The documentation and examples in this repository are under the [MIT License](LICENSE). Use of the Rolev.ai service is
governed by the [Rolev.ai Terms](https://rolev.ai/terms).
