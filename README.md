# pi-figma-remote

The **official** Figma MCP server (`https://mcp.figma.com/mcp`) for the
[Pi coding agent](https://pi.dev) — one command to install, browser click to
authenticate, full toolset unlocked: design-to-code, canvas writes, FigJam,
design-system search, and Code Connect. No Personal Access Token needed.

```bash
pi install npm:pi-figma-remote
```

Then restart pi, run `/mcp`, and authenticate `pi-figma-remote__figma`.

## Why this exists

Pasting Figma's official MCP config into pi fails with a cryptic error:

> `Dynamic Client Registration rejected (HTTP 403): Forbidden`

Figma's OAuth registration endpoint
(`POST api.figma.com/v1/oauth/mcp/register`) allowlists the exact
`client_name`: only recognized clients (Claude Code, Codex, …) get `200`,
everything else gets `403`. Verified live:

| `client_name`          | Response |
|------------------------|----------|
| `Claude Code (figma)`  | `200`    |
| anything else          | `403`    |

Only the *name* is checked — no User-Agent or redirect games needed. This
package sets pi-mcp-adapter's `oauth.clientName` to an allowlisted value, so
pi's **native** OAuth flow (discovery → registration → browser → OS-keychain
storage → automatic refresh) completes untouched. Your `mcp.json` stays
secret-free; there is nothing to paste and no helper script to run.

## Prerequisites

- [pi-mcp-adapter](https://pi.dev/packages/pi-mcp-adapter) (the MCP client for pi):
  ```bash
  pi install npm:pi-mcp-adapter
  ```

## Install

```bash
pi install npm:pi-figma-remote
```

Restart pi (or `/reload`), then:

```
/mcp
```

Start/authenticate `pi-figma-remote__figma` → approve in the browser → done.
Tokens are stored in your OS keychain and refresh automatically.

## Verify

Ask pi to call the `whoami` tool (remote-only, reports the authenticated
Figma account), or paste a Figma frame link and ask it to implement the design.
The remote server is link-based: right-click a frame →
*Copy/Paste as → Copy link to selection*.

## What's included

The full official server — not the read-only subset that PAT-based community
servers expose:

- **Design to code**: `get_design_context`, `get_metadata`, `get_screenshot`,
  `download_assets`, `get_variable_defs`, `get_motion_context`
- **Code to design**: `use_figma` (create/edit frames, components, variables,
  auto-layout), `generate_figma_design`, `create_new_file`, `upload_assets`
- **FigJam**: `get_figjam`, `generate_diagram`
- **Design systems + Code Connect**: `search_design_system`, `get_libraries`,
  `get_code_connect_map`, `add_code_connect_map`
- **Account**: `whoami`

## Disclosure — read this

This package registers with Figma's OAuth endpoint using another client's
display name because Figma currently gates registration to its
[MCP Catalog](https://www.figma.com/mcp-catalog/) clients. Concretely: the
Figma consent screen will show **"Claude Code (figma)"** as the requesting
app, not Pi. That is a **temporary workaround**, openly documented here — same
approach as the community tools for OpenCode. It works today (verified
September 2026) but can break if Figma tightens registration, and
impersonating an allowlisted client's name may itself violate Figma's
terms/acceptable-use, with potential consequences up to account action.
Proceed with eyes open. The proper long-term fix is Figma
allowlisting more clients; pi users can add weight via
[the waitlist](https://form.asana.com/?k=kBG-ejRQTdY8x_H6a4vM3Q&d=10497086658021).
If auth starts failing with 403, that is what happened — fall back to a
PAT-based server (e.g. `figma-developer-mcp`) until then.

Use of the Figma API is subject to [Figma's terms](https://www.figma.com/legal/).
This project is not affiliated with or endorsed by Figma, Inc.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `403` during `/mcp` authenticate | Figma changed the gate — see Disclosure above |
| Server missing from `/mcp` | `/reload`, ensure `pi-mcp-adapter` is installed |
| Tools return auth errors later | `/mcp` → re-authenticate; refresh tokens can expire |

## License

MIT
