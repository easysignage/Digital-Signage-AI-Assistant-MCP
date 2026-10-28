# Use case 1 — Media control: promotion mode

> **Say "promo" and your screens show the promotion. Say "normal" and they go back.**

## The business problem

A store, café, gym, or campus runs a normal content loop all day. Several times a week someone wants to **add** a promotion: a flash sale, happy hour, or an event announcement. Then they want to **remove** it a few hours later.

Doing that by hand means opening the CMS, finding the right media in each playlist, enabling it, publishing to the right screens, and repeating it all in reverse later. Store staff rarely have CMS training. Mistakes are common, like publishing to the wrong screens or forgetting to turn the promo off.

With MCP, the whole procedure becomes **one word in a chat**.

## What the viewer sees

```
NORMAL MODE                               PROMO MODE
┌──────────────────────────────┐         ┌──────────────────────────────┐
│ Menu → Welcome → Menu → News │  promo  │ Menu → 🔥PROMO → Welcome →    │
│ → Menu → Welcome → …         │ ──────▶ │ Menu → 🔥PROMO → News → …     │
└──────────────────────────────┘ ◀────── └──────────────────────────────┘
                                  normal
```

The promotion is shown **in addition to** the normal content. Nothing is removed from the loop. The promo items are simply switched on and off.

---

## How it works technically

### Tags as the contract

We use **one tag, `promo`, in two places**:

| Where | Tag | Meaning |
|---|---|---|
| Media items | `promo` | "This item is part of the promotion." It stays in the playlist and is **disabled** when there's no promotion. |
| Displays | `promo` | "This screen takes part in promotions." Only these screens are published to. |

### The tool sequence

**`promo` → turn the promotion ON**

```
1. enable_media_by_tag      { "tag": "promo" }     # every media item tagged promo becomes enabled
2. publish_display_by_tag   { "tag": "promo" }     # send the updated playlists to every promo screen
3. (optional) list_displays                          # count promo screens online vs offline for the summary
```

**`normal` → turn the promotion OFF**

```
1. disable_media_by_tag     { "tag": "promo" }     # promo items are skipped during playback again
2. publish_display_by_tag   { "tag": "promo" }     # send the change to the same screens
3. (optional) list_displays                          # report offline screens that will update on reconnect
```

**Why publish is always step 2:** enabling or disabling media changes the *plan* in the CMS. **Publishing** sends the plan to the players. Without a publish, the screens keep playing what they had.

**Why the same tag for both:** the publish should reach exactly the screens that can show the promo. Using the same tag makes the setup easy to explain to staff: *"anything tagged promo"*.

### Sequence diagram

```
User            Assistant                       EasySignage MCP            Players
 │  "promo"        │                                  │                        │
 │───────────────▶ │ enable_media_by_tag(promo)       │                        │
 │                 │────────────────────────────────▶ │ enable N media         │
 │                 │ ◀──────────────────────────────── │                        │
 │                 │ publish_display_by_tag(promo)    │                        │
 │                 │────────────────────────────────▶ │ publish ──────────────▶│ download & play
 │                 │ ◀──────────────────────────────── │                        │
 │ ◀────────────── │ "Promotion ON: N media, M screens"│                        │
```

---

## Setup (about 10 minutes)

1. **Create the promotion content.** Upload the image/video. Use the same resolution and orientation as your screens.
2. **Add it to the normal playlist(s)** where you want it to appear. For example, after every second item, or once per loop.
3. **Tag each promo media item `promo`** in EasySignage.
4. **Disable the promo media items.** The default state is "normal".
5. **Tag every participating screen `promo`.**
6. **Publish once** so the screens have the playlist (with the disabled items).
7. **Add the project instructions** from [project-instructions/PROJECT_INSTRUCTIONS.md](../../project-instructions/PROJECT_INSTRUCTIONS.md) to your AI project.

> 💡 **Test first.** Tag one test screen `promo` (and nothing else), run `promo` and `normal`, and check the screen. Then tag the rest.

## Example conversation

```
You:        promo

Assistant:  ✅ Promotion is ON
            • Enabled media tagged "promo"
            • Published to screens tagged "promo" (12 screens: 11 online, 1 offline)
            The offline screen ("Back Entrance") will update when it reconnects.

You:        what's playing on Front Window?

Assistant:  "Front Window" is online and playing the "Store Loop" playlist, which now includes
            the 2 promo items ("Black Friday 30%" and "Black Friday Video").

You:        normal

Assistant:  ✅ Back to normal
            • Disabled media tagged "promo"
            • Published to screens tagged "promo" (12 screens)
```

Natural-language variants the instructions also recognise:

| Turns promo ON | Turns promo OFF |
|---|---|
| `promo`, "promo on", "show the promotion on my screens", "start the sale" | `normal`, "back to normal", "promo off", "stop the promotion", "end the sale" |

---

## Variations

### Several campaigns

Use one tag per campaign and pass the tag name as a parameter:

| Say | Tag used |
|---|---|
| `promo` / `normal` | `promo` |
| `promo breakfast` / `normal breakfast` | `promo-breakfast` |
| `promo black-friday` / `normal black-friday` | `promo-black-friday` |

Media is tagged with the campaign tag. Screens still carry the shared `promo` tag, so they're published to. Update the project instructions to map `promo <name>` → `promo-<name>`.

### Per-region promotions

Add region tags to screens (`sydney`, `melbourne`) and publish by the region tag instead:

```
"promo sydney"  → enable_media_by_tag("promo-sydney") → publish_display_by_tag("sydney")
```

### Time-boxed promotions

Ask: *"promo for 2 hours"*. If your assistant supports scheduled tasks or reminders (e.g. Claude scheduled tasks, Claude Code `/loop`), it can schedule `normal` automatically. Otherwise it reminds you. For promotions on fixed dates, use EasySignage **schedules** instead. The assistant can create a campaign schedule with the `create_schedule` / `create_schedule_event` tools.

---

## Verification and troubleshooting

| Problem | Likely cause | What to ask the assistant |
|---|---|---|
| "Enabled 0 media" | Media isn't tagged `promo`, or there's a typo / different case | "List media tagged promo" |
| Published, but screen unchanged | Screen isn't tagged `promo`, or the promo media isn't in the playlist that screen plays | "What is screen X playing, and does that playlist contain promo media?" |
| Some screens updated, others didn't | Offline screens update when they reconnect | "Which promo screens are offline?" |
| Promo stuck on | Someone enabled it in the CMS, or `normal` wasn't run | "Diagnose: who changed promo media today?" (see [use case 3](../03-diagnose/README.md)) |
| Permission error | The connection has read-only access to media or displays | Reconnect with Full access for Media + Displays ([guide](../../docs/connect-mcp.md#permissions)) |

## Safety rules built into the instructions

- Only the tag named in the instructions is used. The assistant never guesses a tag.
- If a command would publish to **more screens than expected** (e.g. the tag matches 0 or an unusually large number), the assistant stops and asks first.
- The assistant always reports **what it changed** and **how many screens it published to**.
- Every action is recorded in the audit log under the MCP key's name, so you can always trace it.

---

← Back to the [main README](../../README.md) · Next: [Use case 2 — Reporting](../02-reporting/README.md)
