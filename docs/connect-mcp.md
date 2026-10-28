# Connect EasySignage to your AI assistant (MCP)

This guide covers connecting the EasySignage MCP server to **Claude**, **ChatGPT**, **Gemini CLI**, and **Claude Code**.

> 📰 The official walkthrough, with screenshots, is on the EasySignage blog:
> **[Manage digital signage with an AI assistant](https://easysignage.com/blog/26/manage-digital-signage-with-ai-assistant/)**
> Read it alongside this page if you get stuck. It's kept up to date as the AI apps change their menus.

---

## What is MCP, in one paragraph?

The **Model Context Protocol (MCP)** is an open standard for connecting AI assistants to other software. An MCP *server* publishes a list of *tools*, such as `list_displays` or `publish_display_by_tag`. Each tool has a name, a description, and typed inputs. When you ask the assistant something, it decides which tools to call, calls them over HTTPS, and uses the results in its answer. You stay in control: you choose which areas it can access, and every call is recorded in your EasySignage audit log.

## Requirements

- EasySignage **Premium** or **Enterprise** plan
- Signed in to the EasySignage account you want to connect. A connection only applies to the account that's active during setup. Connect each account separately.
- Fewer than 10 API keys on the account. Each MCP connection creates one key.

## Server URL

```
https://mcp.easysignage.com/mcp
```

You can also copy it from **Settings → AI Assistant** in EasySignage.

---

## Claude (claude.ai web & desktop)

### Option 1: Add EasySignage from the Claude directory (easiest)

EasySignage is listed in the Claude connector directory:

1. Open **[claude.ai/directory/easysignage](https://claude.ai/directory/easysignage)**.
2. Select **Connect**.
3. Sign in to EasySignage and **choose access levels** (see [Permissions](#permissions)).
4. In a new chat, select **+ → Connectors** and turn **EasySignage** on.

### Option 2: Add it as a custom connector

1. Open **Customise → Connectors**.
2. Select **+**, then **Add custom connector**.
3. Paste the server URL.
4. Sign in to EasySignage and **choose access levels** (see [Permissions](#permissions)).
5. In a new chat, select **+ → Connectors** and turn **EasySignage** on.

**Claude Team / Enterprise:** an organisation owner adds the connector once under **Organisation settings → Connectors → Add → Custom → Web**. Members then sign in with their own EasySignage account.

## ChatGPT

1. Go to **Settings → Connectors → Advanced settings** and turn on **Developer mode**.
2. Back in **Connectors**, select **Create**.
3. Enter a name (e.g. *EasySignage*), paste the server URL, and choose **OAuth**.
4. Sign in to EasySignage and choose access levels.

**Business / Enterprise:** a workspace admin must first allow custom connectors under **Workspace settings → Permissions & Roles**.

## Gemini CLI

```bash
gemini mcp add --transport http signage https://mcp.easysignage.com/mcp
```

Then start Gemini CLI and run:

```
/mcp auth signage
```

The Gemini web and mobile apps don't accept custom MCP servers yet. In Gemini Enterprise, an admin adds EasySignage as a custom MCP server.

## Claude Code (terminal / IDE)

Claude Code is useful for the **reporting** use case, because it can write files and run on a loop.

```bash
claude mcp add --transport http easysignage https://mcp.easysignage.com/mcp
```

Start `claude`, run `/mcp`, choose **easysignage**, and sign in. Put the project instructions in a `CLAUDE.md` file in your project folder, and Claude Code will follow them automatically.

---

## Permissions

During sign-in, each area (screens, playlists, schedules, folders, media, and so on) can be set to:

| Level | Meaning |
|---|---|
| **No access** | The assistant can't see the area at all |
| **Read only** | It can list and read, but not change anything |
| **Full access** | It can read and make changes (but **never delete** screens, playlists, or media) |

Things to know:

- Areas start at the **highest level the AI app requested**, so review them before confirming.
- If the app requests read-only access, every area is capped at read-only.
- Permissions are **locked once the key is created**. To change them, delete the `MCP:` key in **Settings → API Keys** and reconnect.

**Recommended setup for this tutorial**

| Connection | Use cases | Access |
|---|---|---|
| "Signage Ops" | Media control | Media: Full · Displays: Full · rest: Read only |
| "Signage Reports" | Reporting + Diagnose | Everything: Read only |

Two connections means your 5-minute dashboard job can never change content by accident.

---

## Verify the connection

In a new chat with the connector enabled, ask:

```
List my screens and tell me how many are online.
```

The assistant should call `list_displays` and answer with real data. If it says it has no tools, the connector isn't enabled for that chat.

## Disconnect

Delete the key that starts with **`MCP:`** in **Settings → API Keys**. Access ends immediately. Each key name includes the AI app and the email of the person who approved it, e.g. `MCP: Claude (523e41ef) you@example.com`.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Sign-in page loops or fails | Restart the connection from inside the AI app. Don't reuse an old consent link. |
| Connected to the wrong account | Sign in to the correct EasySignage account first, delete the wrong `MCP:` key, then reconnect. |
| "API key limit reached" | Remove unused keys in Settings → API Keys (max 10 per account). |
| Assistant says "permission denied" for a tool | That area was set to No access or Read only. Delete the key and reconnect with the right levels. |
| Tools don't appear in chat | Turn the connector on for that chat (Claude: **+ → Connectors**). |

---

← Back to the [main README](../README.md)
