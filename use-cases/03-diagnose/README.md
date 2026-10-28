# Use case 3 — Diagnose: audit-log investigation

> **Enterprise plan required.** The audit log is an EasySignage Enterprise feature, so this use case only works on Enterprise accounts. See [pricing](https://easysignage.com/digital-signage-software-pricing/).

> **"Who changed the lobby screen?" "Who deleted the summer video?" Get the answer in seconds, in plain language.**

## The business problem

Something unexpected appears on a screen, or content disappears. The usual investigation means opening the audit log, filtering by hand, decoding resource ids, and piecing together a timeline, often while a store manager is waiting on the phone.

The assistant does this for you. It queries the audit log through MCP, links related entries together, and explains **who did what, when, from where, and how**. It can also suggest a fix.

---

## What's in an audit-log entry

The `list_logs` tool returns entries like this. The example is anonymised:

```json
{
  "datetime": "2026-10-28T04:26:22.132Z",
  "user": "jane@example.com",
  "actor_role": "admin",
  "action": "set source",
  "type": "screen",
  "res_id": "player_ABC123",
  "resource_name": "Lobby",
  "oldValue": "playlist: Main Loop",
  "value": "name: Lobby, playlist: Test Playlist",
  "source": "ui",
  "api_key_name": "",
  "request_outcome": "success",
  "client_ip": "203.0.113.25",
  "city": "Sydney",
  "country": "AU",
  "request_id": "af1b29eb07897ed927a0413599474b79"
}
```

| Question | Field(s) |
|---|---|
| **Who?** | `user` (email, or `open-api:<key prefix>` for API keys), `actor_role`, `api_key_name` (e.g. `MCP: Claude (…) jane@example.com` for AI changes) |
| **Did what?** | `action` (`add`, `update`, `delete`, `archive`, `publish`, `set source`, `login`…) |
| **To what?** | `type` (`screen`, `playlist`, `media`, `region`, `api_key`, `access`…), `res_id`, `resource_name`, `parent_type` / `parent_res_id` |
| **What changed?** | `oldValue` → `value`, `changed_fields` |
| **When?** | `datetime` (UTC), `local_datetime` when available |
| **From where?** | `client_ip`, `city`, `country` |
| **How?** | `source`: `ui` (web app), `open_api` (API key / MCP), `webhook`, `internal` (platform), `anonymous` |
| **Did it work?** | `request_outcome` |
| **Related entries** | `request_id`: every entry written by the same request |

### Filters the assistant can use

`list_logs` accepts: `action`, `type`, `user`, `res_id`, `source`, `authored_by` (`server`/`client`), `request_id`, `from`, `to` (ISO date-times, inclusive), and `limit`/`offset` for paging (up to 500 per page).

---

## How the assistant investigates

```
1. Understand the question      → which screen/media/playlist? which time window? which action?
2. Resolve names to ids         → list_displays / list_media / list_playlists to find res_id
3. Query narrowly first         → list_logs(res_id=…, from=…, to=…)
4. Widen if needed              → by type/action/user, or a longer window; page with offset
5. Follow the thread            → request_id ties related entries together;
                                  a "set source" is usually followed by a "publish"
6. Explain                      → timeline table + plain-language cause + suggested fix
```

The assistant **only reads** in this use case. If it suggests a fix (e.g. "set the source back and publish"), it asks before doing anything. Deletions can't be undone through MCP (the assistant can't delete or restore), so it points you to the CMS for that.

---

## Investigation playbooks

### A. "Who published to this screen?"

```
Who published to the "Lobby" screen in the last 24 hours?
```

Tool calls:

```
list_displays                                   → find "Lobby" → res_id = player_ABC123
list_logs { res_id: "player_ABC123", action: "publish", from: "<now-24h>" }
list_logs { res_id: "player_ABC123", action: "set source", from: "<now-24h>" }   # what was published
```

Answer format:

```
Lobby (player_ABC123), last 24 h — 2 publishes

Time (local)   Who                  What                                          Via
08:41          jane@example.com     set source: Main Loop → Test Playlist         Web app · Sydney, AU
08:41          jane@example.com     publish                                       Web app · Sydney, AU
14:02          MCP: Claude (…) tom@  publish (promo tag)                          AI assistant (API)

Summary: Jane switched Lobby to "Test Playlist" at 08:41 and published. At 14:02, Tom's AI
assistant published the promo to all promo-tagged screens, which included Lobby.
```

### B. "Who deleted something?"

```
Who deleted the "Summer Sale" video?
```

```
list_logs { action: "delete", type: "media", from: "<now-30d>" }
→ look for resource_name / oldValue containing "Summer Sale"
```

`oldValue` on a delete holds a snapshot of the deleted item, which is useful to confirm it's the right one and to recreate it.

### C. "What changed on this playlist?"

```
What changed in the "Store Loop" playlist this week, and by whom?
```

```
list_playlists → res_id
list_logs { res_id: "<playlist id>", from: "<monday>" }
list_logs { type: "media", from: "<monday>" }  → keep entries whose parent_res_id = playlist id
```

### D. "What did the AI do?"

Every MCP change is made with that connection's API key, so it's easy to audit:

```
Show me every change made through the API or AI assistants today.
```

```
list_logs { source: "open_api", from: "<today 00:00>" }
```

Group the results by `api_key_name` (e.g. `MCP: Claude (523e41ef) tom@example.com`).

### E. Security review

```
Give me a security summary for the last 7 days: logins, new or deleted API keys,
and any failed requests.
```

```
list_logs { type: "access", action: "login", from: "<now-7d>" }
list_logs { type: "api_key", from: "<now-7d>" }
list_logs { from: "<now-7d>" } → keep request_outcome != "success"
```

Watch for logins from unexpected countries, new API keys nobody recognises, and many failed requests.

### F. "Why is the promotion still showing?"

```
The promo is still on the Perth screens. What happened?
```

```
list_logs { type: "media", action: "update", from: "<today>" }   → enable/disable of promo-tagged media
list_logs { action: "publish", from: "<today>" }                → was there a publish after the disable?
```

A common finding is that someone disabled the promo media in the CMS but never published. The fix is a single `publish_display_by_tag("promo")`.

---

## Prompt library

| Ask | Good for |
|---|---|
| "What happened to screen *X* today?" | Full timeline for one screen |
| "Who published to screens tagged *sydney* yesterday?" | Region-level change tracking |
| "Who deleted media in the last 30 days?" | Missing content |
| "List every change by *jane@example.com* this week." | Activity of one user |
| "Show changes made through the API/AI since Monday." | AI/automation audit |
| "Were there any failed requests today?" | Errors and permission problems |
| "Who created or removed API keys this month?" | Access governance |
| "Explain request *request_id*." | Everything one action did |

---

## Tips for accurate answers

- **Give a time window** ("today", "since 9 am", "last 7 days"). The assistant converts it to `from`/`to` in UTC and shows times back in your timezone.
- **Use exact names** for screens and media. The assistant resolves them to ids first, which makes the log query precise.
- **Large accounts:** logs are paged (up to 500 per call). The assistant should say when it stopped early, e.g. "showing the latest 500 of more".
- **Privacy:** audit logs contain emails and IP addresses. Share answers only with the people who need them. Don't paste them into public channels.

---

← Back to the [main README](../../README.md) · Previous: [Use case 2](../02-reporting/README.md)
