# Digital Signage AI Assistant with MCP (EasySignage)

Manage your digital signage screens by **chatting with an AI assistant**, using EasySignage's **MCP server** (Claude, ChatGPT, or Gemini).

This repository is a hands-on tutorial. It shows three practical use cases you can set up in about 15 minutes. You also get a ready-made **project instruction set** to copy into your AI assistant.

📰 **Read the overview on our blog:** [easysignage.com/blog/26/digital-signage-mcp-ai-assistant-use-cases](https://easysignage.com/blog/26/digital-signage-mcp-ai-assistant-use-cases/)

---

## What we will build

| # | Use case | What you say | What happens |
|---|----------|--------------|--------------|
| 1 | **[Media control: promotion mode](#1-media-control--promotion-mode)** | `promo`, or "show the promotion on my screens" | The assistant enables every media item tagged `promo` and publishes to every screen tagged `promo`. Screens play the promotion **in addition to** their normal content. |
|   |  | `normal`, or "back to normal" | The assistant disables the `promo` media and publishes again. Screens go back to their normal content only. |
| 2 | **[Reporting: live player dashboard](#2-reporting--live-player-status-dashboard)** | "Build my player dashboard" / "Refresh the dashboard" | The assistant reads the status of every player (online/offline, power, software version, what it's playing). It writes that to a dashboard web page that refreshes every 5 minutes. **The dashboard itself runs on a digital signage screen.** |
| 3 | **[Diagnose: audit-log investigation](#3-diagnose--audit-log-investigation)** (Enterprise plan) | "Who changed what's on the lobby screen yesterday?" / "Who deleted the summer video?" | The assistant searches the audit logs and reports a clear timeline: who, what, when, from where, and through which channel (web app, API, or AI). |

Then we put it all together:

- **[Connect the MCP server](#step-0--connect-easysignage-to-your-ai-assistant)** (one-time setup)
- **[Copy-paste project instructions](#all-three-in-one-project-copy-paste-instructions)** that teach the assistant all three tasks

```
                ┌──────────────────────┐
  "promo"  ───▶ │   AI assistant       │      MCP (HTTPS + OAuth)      ┌────────────────────┐
  "normal" ───▶ │ (Claude / ChatGPT /  │ ────────────────────────────▶ │ EasySignage MCP    │
  "report" ───▶ │  Gemini) + project   │ ◀──────────────────────────── │ server             │
  "who…?"  ───▶ │  instructions        │   tool calls & results        │ mcp.easysignage.com│
                └──────────────────────┘                               └─────────┬──────────┘
                                                                                 │
                                                       media · displays · logs · │ publish
                                                                                 ▼
                                                                    ┌────────────────────────┐
                                                                    │  Your screens/players  │
                                                                    └────────────────────────┘
```

---

## Repository layout

```
.
├── README.md                          ← you are here (overview + walkthrough)
├── docs/
│   ├── connect-mcp.md                 ← detailed MCP connection guide (Claude, ChatGPT, Gemini)
│   └── concepts.md                    ← EasySignage + MCP concepts in plain language
├── project-instructions/
│   └── PROJECT_INSTRUCTIONS.md        ← copy-paste this into your AI project
└── use-cases/
    ├── 01-media-control/README.md     ← promotion on/off, in depth
    ├── 02-reporting/
    │   ├── README.md                  ← dashboard, in depth
    │   └── dashboard/
    │       ├── index.html             ← the signage-ready dashboard (single file, no build step)
    │       └── status.json            ← sample data the dashboard reads
    └── 03-diagnose/README.md          ← audit-log investigations, in depth
```

---

## Before you start

You need:

1. An **EasySignage account** on the **Premium or Enterprise** plan. MCP access is included in these plans.
   - **Use case 3 (Diagnose) needs the Enterprise plan**, because it reads the audit log, which is an Enterprise feature. Use cases 1 and 2 work on Premium.
2. An **AI assistant that supports custom MCP connectors**: Claude (web, desktop, or Claude Code), ChatGPT (Developer mode), or Gemini CLI.
3. A few minutes to **tag your content and screens** (explained in each use case).

New to these terms? [docs/concepts.md](docs/concepts.md) explains *display*, *playlist*, *media*, *tag*, *publish*, and *MCP* in one page.

---

## Step 0 — Connect EasySignage to your AI assistant

MCP (Model Context Protocol) is an open standard that lets an AI assistant safely call tools in another system. The EasySignage MCP server gives your assistant tools such as `list_displays`, `enable_media_by_tag`, `publish_display_by_tag`, and `list_logs`.

**Server URL**

```
https://mcp.easysignage.com/mcp
```

You can also copy it from **Settings → AI Assistant** in EasySignage.

**Quickest setup (Claude): from the Claude directory.** Open **[claude.ai/directory/easysignage](https://claude.ai/directory/easysignage)**, select **Connect**, then sign in to EasySignage and choose access levels. No URL to paste.

**Quick setup (Claude, as a custom connector):** Customise → Connectors → **+** → *Add custom connector* → paste the URL → sign in and choose access levels → in a new chat, **+** → Connectors → turn **EasySignage** on.

**Quick setup (ChatGPT):** Settings → Connectors → Advanced settings → turn on **Developer mode** → Connectors → **Create** → name it, paste the URL, choose **OAuth** → sign in and choose access levels.

**Quick setup (Gemini CLI):**

```bash
gemini mcp add --transport http signage https://mcp.easysignage.com/mcp
```

Then start Gemini CLI and run `/mcp auth signage`.

**Claude Code (terminal):**

```bash
claude mcp add --transport http easysignage https://mcp.easysignage.com/mcp
```

Then run `/mcp` inside Claude Code to sign in.

### Choosing permissions for these use cases

During sign-in you set each area to **No access**, **Read only**, or **Full access**. For this tutorial:

| Area | Media control | Reporting | Diagnose (Enterprise) |
|------|:---:|:---:|:---:|
| Media | Full | — | Read |
| Displays (screens) | Full (to publish) | Read | Read |
| Audit logs* | — | — | Read |
| Everything else | Optional | Optional | Optional |

\* If your consent screen lists a separate logs/audit area, set it to **Read only**. Diagnosis only needs to read.

> Permissions are **locked** once the connection is created. To change them, delete the key that starts with `MCP:` in **Settings → API Keys** and connect again.
> The assistant **cannot delete** screens, playlists, or media, even with full access.

📖 **Full step-by-step guide with screenshots and troubleshooting:** [docs/connect-mcp.md](docs/connect-mcp.md)
📖 **Official EasySignage article:** [Manage digital signage with an AI assistant](https://easysignage.com/blog/26/manage-digital-signage-with-ai-assistant/)

---

## 1. Media control — promotion mode

**Goal:** turn a promotion on across your screens with one word, and turn it off with another, without opening the CMS.

### The idea

Tags let you label things in EasySignage. We use **one tag, `promo`, in two places**:

- **On media:** the promotional images/videos inside your normal playlists are tagged `promo`. They stay in the playlist but are **disabled** most of the time.
- **On screens:** every screen that should show the promotion is tagged `promo`.

| You say | Assistant calls | Result on screen |
|---|---|---|
| `promo` | `enable_media_by_tag(tag="promo")` → `publish_display_by_tag(tag="promo")` | Normal content **+** promotion |
| `normal` | `disable_media_by_tag(tag="promo")` → `publish_display_by_tag(tag="promo")` | Normal content only |

Why this works: a disabled media item stays in its playlist but is skipped during playback. Enabling it puts it back into the loop. **Publishing** is what sends the change to the players, so the assistant always publishes after enabling or disabling.

### One-time setup

1. Upload your promotion media. Add it to the playlist(s) your screens already play, wherever it should appear in the loop.
2. Tag each promotion media item `promo`.
3. Disable those items (so screens start in *normal* mode).
4. Tag the screens that should show the promotion `promo`.

### Try it

```
You:        promo
Assistant:  Promotion ON. Enabled 3 media items tagged "promo" and published to 12 screens
            tagged "promo". Screens will show the promotion after their next sync (usually within a minute).

You:        back to normal
Assistant:  Promotion OFF. Disabled 3 "promo" media items and published to 12 screens. Screens are
            back to normal content.
```

📖 **In depth** (multiple campaigns, per-region promos, scheduling, verification, troubleshooting): [use-cases/01-media-control](use-cases/01-media-control/README.md)

---

## 2. Reporting — live player status dashboard

**Goal:** a full-screen dashboard showing the health of every player. It updates every 5 minutes and is designed to **run on a digital signage screen** in your office, NOC, or store back room.

### How it fits together

```
 every 5 min                      writes                        fetches every 60 s
┌────────────────┐  list_displays  ┌──────────────┐   hosted at   ┌──────────────────────────┐
│ AI assistant   │ ──────────────▶ │ status.json  │ ────────────▶ │ dashboard/index.html     │
│ (scheduled     │   via MCP       │ (summary of  │  any static   │ added to EasySignage as a │
│  task / loop)  │                 │  all players)│  web host     │ Web/URL media → on screen │
└────────────────┘                 └──────────────┘               └──────────────────────────┘
```

1. The assistant calls `list_displays` (read-only) and summarises every player: online/offline, power, OS, software version, current playlist, licence expiry.
2. It writes the result to `status.json` in the format the dashboard expects.
3. [`dashboard/index.html`](use-cases/02-reporting/dashboard/index.html) is a single static file (no build, no API keys in the browser). It reads `status.json` every 60 seconds and shows:
   - KPI tiles: total players, online, offline, powered off, licence issues
   - An "attention needed" list (offline players first, then screens powered off, then licence problems)
   - A full player table with status, platform, version, and current content
   - A **"data is stale"** warning if no update arrives for 15 minutes
4. You add the dashboard URL to EasySignage as a **Web page** media item, put it in a playlist, and assign it to your ops screen. The assistant can do this step for you.

### Try it

Open `use-cases/02-reporting/dashboard/index.html` locally (it ships with sample data), then ask your assistant:

```
Build my player status report and save it as status.json for the dashboard.
```

📖 **In depth** (scheduling every 5 min with Claude Code / scheduled tasks, hosting options, the JSON schema, putting it on a screen, security): [use-cases/02-reporting](use-cases/02-reporting/README.md)

---

## 3. Diagnose — audit-log investigation

> **Enterprise plan required.** This use case reads the EasySignage audit log, which is included in the Enterprise plan only.

**Goal:** answer "what happened?" in seconds. For example: who published to a screen, who changed what it plays, who deleted something.

### What the assistant can see

Every change in EasySignage is written to the **audit log**. The `list_logs` tool can filter by:

| Filter | Example |
|---|---|
| `action` | `publish`, `set source`, `add`, `update`, `delete`, `login` |
| `type` | `screen`, `playlist`, `media`, `region`, `api_key`, `access` |
| `user` | `jane@example.com`, or `open-api:<key prefix>` for API/AI changes |
| `from` / `to` | ISO date-times, e.g. `2026-10-27T00:00:00Z` |
| `res_id` | the id of a screen, playlist, or media item |
| `source` | `ui` (web app), `open_api` (API/MCP), `webhook`, `internal` |

Each entry tells you **who** (`user`, `actor_role`, `api_key_name`), **what** (`action`, `type`, `resource_name`, `oldValue` → `value`), **when** (`datetime`), **where from** (`client_ip`, `city`, `country`), **how** (`source`), and whether it **succeeded** (`request_outcome`).

### Try it

```
You:        Why is the "Lobby" screen showing the wrong playlist since this morning?

Assistant:  Here's what happened to "Lobby" (player_ABC123) today:

            08:41  jane@example.com  set source   playlist: "Main Loop" → "Test Playlist"   (web app, Sydney AU)
            08:41  jane@example.com  publish      Lobby                                      (web app, Sydney AU)

            Cause: Jane switched Lobby to "Test Playlist" at 08:41 and published it. No changes since.
            Fix: set the source back to "Main Loop" and publish. Want me to do that?
```

📖 **In depth** (investigation playbooks, prompt library, reading `oldValue`/`value`, spotting AI/API changes, security reviews): [use-cases/03-diagnose](use-cases/03-diagnose/README.md)

---

## All three in one project: copy-paste instructions

Most AI assistants let you create a **Project** (Claude Projects, ChatGPT Projects/GPTs, Gemini Gems, or a `CLAUDE.md` in Claude Code). It has standing instructions that apply to every chat.

1. Create a new project, e.g. **"Signage Ops"**.
2. Open **[project-instructions/PROJECT_INSTRUCTIONS.md](project-instructions/PROJECT_INSTRUCTIONS.md)** and copy everything inside the code block.
3. Paste it into the project's **Instructions** / **Custom instructions** field.
4. Make sure the **EasySignage connector is enabled** for chats in that project.
5. Type `promo`, `normal`, `report`, or ask a "who/what/when" question.

The instructions include the trigger words, the exact tool sequence for each task, safety rules (confirm before broad changes, never guess tags), and the output formats.

---

## Safety and good practice

- **Least privilege.** Give the connection only the access each task needs. Use a separate read-only connection for reporting and diagnosis if you can.
- **Publishing is the "go live" button.** Enabling or disabling media changes the plan. Publishing sends it to screens. The instructions tell the assistant to report how many screens it published to.
- **Tags are the contract.** Pick tag names once (e.g. `promo`) and keep them consistent. A typo means nothing happens, and the assistant will tell you it found zero items.
- **Everything is audited.** Changes made through MCP appear in the audit log with the connection's API key name (`MCP: Claude (…) you@example.com`). Use case 3 can always tell you what the AI did.
- **Disconnect anytime.** Delete the `MCP:` key in **Settings → API Keys**. Access stops immediately.
- **Limits.** An account can hold up to 10 API keys. Each connection applies to the account that was active when you set it up.

---

## FAQ

**Does the AI keep access to my account forever?**
No. Access lasts until you delete the `MCP:` key in Settings → API Keys.

**Can the assistant delete my content?**
No. The EasySignage MCP server doesn't allow deleting screens, playlists, or media.

**My screens didn't change after "promo".**
Check that (1) media items are tagged exactly `promo`, (2) screens are tagged `promo`, (3) the promo media is inside a playlist those screens play. Offline screens pick up the change when they reconnect. On the Enterprise plan you can also ask: *"Diagnose: did the promo publish succeed?"*

**Can I use a different word than "promo"?**
Yes. Change the tag and trigger words in the project instructions, e.g. `sale` / `end sale`, or `breakfast` / `lunch`.

**Which plan do I need?**
Premium or Enterprise for MCP access (use cases 1 and 2). Enterprise for use case 3, because the audit log is an Enterprise feature. See [pricing](https://easysignage.com/digital-signage-software-pricing/).

**Which AI assistant is best?**
All three work. For the 5-minute dashboard refresh, use an assistant that can run scheduled or looping tasks, such as Claude Code (`/loop`), Claude scheduled tasks, or a cron job running a CLI assistant.

---

## Learn more

- 📰 [MCP use cases for digital signage (EasySignage blog)](https://easysignage.com/blog/26/digital-signage-mcp-ai-assistant-use-cases/)
- 📰 [Manage digital signage with an AI assistant (EasySignage blog)](https://easysignage.com/blog/26/manage-digital-signage-with-ai-assistant/)
- 🌐 [easysignage.com](https://easysignage.com)
- 📘 [Model Context Protocol](https://modelcontextprotocol.io)

Contributions and new use-case ideas are welcome. Open an issue or pull request.

---

📰 **Full write-up on our blog:** [easysignage.com/blog/26/digital-signage-mcp-ai-assistant-use-cases](https://easysignage.com/blog/26/digital-signage-mcp-ai-assistant-use-cases/)
