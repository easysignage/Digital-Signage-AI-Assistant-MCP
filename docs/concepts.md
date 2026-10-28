# Concepts in plain language

A one-page glossary for everything this tutorial uses.

## EasySignage

| Term | What it is | Why it matters here |
|---|---|---|
| **Display** (screen, player) | One physical screen and the player device driving it (Android, webOS, Tizen, Windows…). | Reporting shows the status of each one. Media control publishes to them. |
| **Playlist** | An ordered set of media that a display plays in a loop. | Your promo media lives inside your normal playlists. |
| **Media** | An image, video, web page, or app inside a playlist. | Promo media is enabled/disabled by tag. The dashboard is added as a web-page media item. |
| **Enabled / disabled media** | A disabled item stays in the playlist but is skipped during playback. | This is how "promo" and "normal" switch without rebuilding playlists. |
| **Tag** | A free-form label on media or displays (`promo`, `sydney`, `store-042`). | Tags let one command target many items at once. |
| **Publish** | Sends the current plan (playlists, media, settings) to the players. | **Nothing changes on screen until you publish.** Offline displays get the update when they reconnect. |
| **Source** | What a display is set to play: a playlist, a schedule, a channel… | The diagnose use case looks for `set source` changes. |
| **Audit log** | A record of every change: who, what, when, from where, and through which channel. | The diagnose use case reads it. Enterprise plan only. |

## MCP and AI

| Term | What it is |
|---|---|
| **MCP** (Model Context Protocol) | An open standard that lets AI assistants call tools in other systems. |
| **MCP server** | The service that exposes the tools. For EasySignage it's `https://mcp.easysignage.com/mcp`. |
| **Tool** | One action the assistant can take, e.g. `list_displays`, `enable_media_by_tag`, `publish_display_by_tag`, `list_logs`. |
| **Connector** | What Claude/ChatGPT call an MCP server you've added. |
| **Project instructions** | Standing instructions the assistant follows in every chat in a project. They turn "promo" into a defined procedure. |
| **Scoped API key** | Each MCP connection gets its own EasySignage API key, limited to the permissions you chose. Delete it to revoke access. |

## Tools used in this tutorial

| Tool | Read/Write | Used in |
|---|---|---|
| `enable_media_by_tag` | Write | Media control |
| `disable_media_by_tag` | Write | Media control |
| `publish_display_by_tag` | Write | Media control |
| `list_displays` | Read | Media control (verify), Reporting |
| `get_display` | Read | Reporting, Diagnose |
| `list_media` / `get_media` | Read | Media control (verify), Diagnose |
| `create_media_from_url` | Write | Reporting (add the dashboard to a playlist, optional) |
| `list_logs` | Read | Diagnose |
| `search_docs` | Read (product docs only) | Any "how does EasySignage…" question |

Tool names can differ slightly between assistants. Some show them with a prefix, but the assistant chooses the right one by its description.

---

← Back to the [main README](../README.md)
