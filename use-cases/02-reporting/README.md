# Use case 2 — Reporting: live player status dashboard

> **The AI assistant collects player status every 5 minutes. A dashboard shows it on a screen in your office.**

## The business problem

When you run tens or hundreds of screens, the first question every morning is *"is everything playing?"*. Usually someone logs in to the CMS and scrolls through the player list. Problems are found late, often by a customer or store manager.

A wall-mounted status board fixes that. Here the **AI assistant is the data pipeline**: it reads player status through MCP, shapes it into a small JSON file, and a static web page shows it. You get a working operations dashboard without a backend, a database, or API code.

## What it looks like

```
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│ Player Status   Demo Retail Group                                    09:04               │
│                                                                      Updated 4 min ago   │
├──────────────┬──────────────┬──────────────┬──────────────┬──────────────────────────────┤
│ TOTAL  12    │ ONLINE  9    │ OFFLINE  3   │ POWERED OFF 1│ LICENCE ISSUES  1            │
│              │ 75% avail.   │              │              │                              │
├──────────────┴───────────┬──┴──────────────┴──────────────┴──────────────────────────────┤
│ Needs attention (5)      │ All players                                     page 1 of 2   │
│ ▌Adelaide – Showroom     │ ● Offline  Adelaide – Showroom   Showroom (playlist)  tizen  │
│ ▌ Offline · was playing… │ ● Offline  Brisbane – Reception  Corporate (playlist) win…   │
│ ▌Brisbane – Reception    │ ● Offline  Sydney – Back Entr…   Store Loop (playlist) webos │
│ ▌Melbourne – Café Menu   │ ● Screen off  Melbourne – Café…  Café Menu (playlist)        │
│ ▌ Online, screen off     │ ● Online   Melbourne – Gym Floor Fitness Loop (playlist)     │
└──────────────────────────┴───────────────────────────────────────────────────────────────┘
```

Open [`dashboard/index.html`](dashboard/index.html) in a browser to see it with the sample data. Because the sample's timestamp is in the past, you'll also see the **"data is stale"** warning, which shows how the board reacts when the refresh stops.

> Opening the file directly (`file://`) blocks `fetch` in some browsers. Serve the folder instead:
>
> ```bash
> cd use-cases/02-reporting/dashboard && python3 -m http.server 8080
> ```
>
> Then open http://localhost:8080.

---

## Architecture

```
 ┌───────────────────────┐   1. list_displays (read-only)   ┌────────────────────────┐
 │ AI assistant          │ ───────────────────────────────▶ │ EasySignage MCP server │
 │ runs every 5 minutes  │ ◀─────────────────────────────── │                        │
 │ (scheduled task/loop) │        player summaries          └────────────────────────┘
 └──────────┬────────────┘
            │ 2. write status.json (schema below)
            ▼
 ┌───────────────────────┐   3. fetch every 60 s   ┌──────────────────────────────────┐
 │ Static web host       │ ◀────────────────────── │ dashboard/index.html             │
 │ index.html +          │                         │ running in a Web page media item │
 │ status.json           │                         │ on a signage player              │
 └───────────────────────┘                         └──────────────────────────────────┘
```

Design choices:

- **No secrets in the browser.** The page only reads a JSON file. The EasySignage credentials stay inside the AI assistant's MCP connection.
- **Read-only connection.** The reporting job only needs `list_displays` (and optionally `get_display`). Give it a read-only connection so the job can never change content.
- **Two refresh speeds.** Data is refreshed every **5 minutes** by the assistant. The page re-reads the file every **60 seconds**, so a new file appears within a minute.
- **Fail-visible.** If no new data arrives for 15 minutes, the page shows a yellow banner instead of quietly showing old numbers.
- **Built for signage.** Large type, dark theme (less glare, less burn-in), no scrolling (long lists page automatically every 10 s), and a portrait layout for vertical screens.

---

## Step 1 — Generate `status.json` with the assistant

Ask, in a chat that has the EasySignage connector enabled:

```
report
```

or in full:

```
Build my player status report: call list_displays (page through all players), and write
status.json in the dashboard format from the project instructions.
```

The assistant:

1. Calls `list_displays` with `limit: 500`. If it gets a full page, it calls again with `offset` until it has every player.
2. Keeps only the summary fields the dashboard needs.
3. Writes `status.json` (in Claude Code it saves the file directly; in a chat app it gives you the file to download).
4. Replies with a short text summary, e.g. *"12 players: 9 online, 3 offline (Adelaide – Showroom, Brisbane – Reception, Sydney – Back Entrance)."*

### `status.json` schema

```jsonc
{
  "generatedAt": "2026-10-28T09:00:00Z",   // ISO-8601 UTC time when the assistant built the file (required)
  "account": "Demo Retail Group",          // shown next to the title (optional)
  "refreshMinutes": 5,                     // shown in the footer (optional)
  "summary": {                              // for humans and other tools; the page recalculates it from "displays"
    "total": 12, "online": 9, "offline": 3, "poweredOff": 1, "licenceIssues": 1
  },
  "displays": [                             // one entry per player (required)
    {
      "id": "player_demo01",                // from list_displays
      "name": "Sydney – Front Window",
      "online": true,                       // connected to EasySignage right now
      "power": true,                        // screen powered on
      "osPlatform": "android",              // android | webos | tizen | windows | …
      "softwareVer": 275,
      "type": "horizontal",                 // horizontal | vertical
      "sourceType": "playlist",             // playlist | schedule | channel | …
      "sourceName": "Store Loop",           // what the player is set to play
      "tags": ["promo", "sydney"],
      "expired": false                      // licence expired
    }
  ]
}
```

All `displays[]` fields map one-to-one to the summary fields returned by `list_displays`, so the assistant doesn't need to calculate anything.

---

## Step 2 — Refresh every 5 minutes

A chat window doesn't run on its own, so pick one of these options:

### Option A — Claude Code `/loop` (simplest to try)

In a folder that contains `status.json` and a `CLAUDE.md` with the project instructions:

```
/loop 5m report and save status.json, then upload it to the dashboard host
```

Claude Code repeats the task every 5 minutes for as long as the session is open. Good for demos and short events.

### Option B — Scheduled task (keeps running)

Use your assistant's scheduler (for example Claude **scheduled tasks** / routines) with the prompt:

```
Every 5 minutes: run the "report" task from the project instructions and publish status.json
to <your host>.
```

### Option C — Cron + headless assistant (production)

Run a CLI assistant non-interactively from cron on a small server:

```bash
# crontab -e
*/5 * * * * cd /opt/signage-dashboard && claude -p "report: write status.json" --allowedTools "mcp__easysignage__list_displays,Write" && ./upload.sh
```

`upload.sh` copies `status.json` to your host (e.g. `aws s3 cp`, `rsync`, `scp`, or a git push for GitHub Pages).

> **Cost tip:** a 5-minute refresh is 288 runs per day. If your fleet changes slowly, 10–15 minutes is usually enough. The dashboard's stale threshold can be changed with `?stale=30`.

---

## Step 3 — Host the dashboard

The dashboard is two static files. Put `index.html` and `status.json` in the same folder on any HTTPS host:

| Host | Notes |
|---|---|
| GitHub Pages / Netlify / Cloudflare Pages | Free and simple. Make the repo private if your player names are sensitive (Pages on private repos needs a paid GitHub plan). |
| Amazon S3 / Azure Blob static website | Easy to update `status.json` from cron with the CLI. |
| Internal web server | Best when the board is only shown inside your network. |

If the JSON is stored somewhere else, point the page at it:

```
https://dash.example.com/index.html?src=https://data.example.com/signage/status.json
```

(The data host must allow CORS from the dashboard's origin.)

### URL options

| Parameter | Default | Meaning |
|---|---|---|
| `src` | `status.json` | Where to read the data |
| `poll` | `60` | Seconds between data reloads |
| `stale` | `15` | Minutes before the stale-data warning appears |
| `page` | `10` | Seconds per page when there are more players than fit on screen |

---

## Step 4 — Put it on a screen (the assistant can do this too)

1. In EasySignage, add a **Web page** media item with the dashboard URL.
2. Add it to a playlist (e.g. *Ops Dashboard*) with a long duration. It refreshes its own data.
3. Assign the playlist to your ops screen and **publish**.

Or ask the assistant, using a connection with write access:

```
Create a web page media item for https://dash.example.com/ in a new playlist called
"Ops Dashboard", assign it to the screen "Adelaide – Ops Room", and publish it.
```

The assistant uses `create_media_from_url`, `create_playlist`, `update_display_source`, and `publish_display`.

---

## Ideas to extend it

- **Alert as well as show:** have the scheduled task also send a message (email, Slack, Teams) when a player has been offline for two runs in a row.
- **History:** append each run to `history.jsonl` and add an uptime-per-day chart.
- **Per-region boards:** filter by tag (`?tag=sydney` is a small change in `render()`).
- **Proof-of-play:** add a daily summary from `export_proof_of_play`.

---

← Back to the [main README](../../README.md) · Previous: [Use case 1](../01-media-control/README.md) · Next: [Use case 3 — Diagnose](../03-diagnose/README.md)
