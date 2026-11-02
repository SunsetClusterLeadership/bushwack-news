# Posting to the Bushwack TD mailbox

A guide to writing and publishing mailbox posts and gifts. The full technical reference is
`docs/ops/mailbox-feed.md` in the game repository; this is the working summary.

## What the feed is

`feed.json` in this repository is the game's mailbox. GitHub Pages serves it at
**https://sunsetclusterleadership.github.io/bushwack-news/feed.json**, and the game downloads it
at launch and whenever the mailbox is opened, at most once every 10 minutes.

- `posts` fill the **News** tab. `rewards` fill the **Rewards** tab (gifts players claim).
- The game also ships a copy of the feed inside the build (the *bundled* feed), so the mailbox
  works offline. The online feed is laid over it **by id**: an online entry with the same id
  replaces the bundled one, and new ids are added.
- If a download fails, or the file is broken, the game keeps the last good download, or the
  bundled feed if there is none. A broken file never crashes the game, but nothing new shows.
- Changing `feed.json` never needs a new build. Pages takes a minute or two to publish a commit.
- Everything here is public and shown to players.

## Post fields (News)

| Field | Required | What it is |
|---|---|---|
| `id` | yes | Unique text, never reused (see below). |
| `kind` | yes | `update`, `event` or `notice`. Picks the built-in banner. |
| `title` | yes | The row title and the opened post's title. |
| `date` | yes | `YYYY-MM-DD`. Shown on the row; the list is newest first. Does not hide or show the post. |
| `start`, `end` | no | UTC times. Hidden before `start` and from `end` on. |
| `banner` | no | `update`, `event` or `notice` (built-in art), or the full `https://` address of a PNG in `images/`. Defaults to the kind. |
| `headline` | no | Big white text over the left of the banner in the opened post. |
| `body` | yes | A list of lines. A line starting with `• ` is a bullet; `""` is a paragraph gap. |
| `gift` | no | A list of reward items. Adds a Claim row to the post. |
| `gift_caption` | no | The words next to the gift. Default "A gift for you!". |

## Reward fields (Rewards)

| Field | Required | What it is |
|---|---|---|
| `id` | yes | Unique text, never reused, and not the same as any post's id. |
| `title` | yes | The row title. |
| `reason` | no | The line under it. |
| `items` | yes | A list of reward items. The row shows the first, with "+N" for the rest. |
| `start` | no | UTC time. Hidden before it. |
| `expires` | yes | UTC time. The row counts down to it. After it, the gift shows only to players who claimed it (as "Claimed") for one more day. |

Each reward, and each post's gift, can be claimed **once per player, ever**.

## Item types and caps

- `{"type": "gems", "amount": 250}`
- `{"type": "petals", "amount": 1000}`
- `{"type": "key", "tier": "common", "amount": 1}`. The tier is `common`, `rare`, `epic`,
  `legendary` or `spell`. A key opens one chest of that tier.

Amounts are whole numbers above 0. One item never grants more than **50,000 gems**, **100,000
petals** or **20 keys**: anything above is granted at the cap.

## Never reuse an id

Each player's "read" and "claimed" are remembered **by id, forever**. A reused id shows as already
read, and a gift with a reused id cannot be claimed by anyone who claimed the old one. Use a fresh,
descriptive id every time, e.g. `event-halloween-2026`, `welcome-testers-2026-11`. Never delete an
entry to reuse its id, and record every id in the Post history below.

## Hiding a post, including the bundled ones

To take a post down, set its `end` to a time in the past (e.g. `"end": "2026-11-01T00:00:00Z"`)
and keep the entry. That works for every post, and it is the **only** way to hide a bundled post:
the bundled posts live inside the game, so deleting them from this file does not hide them.

## Times

UTC, written `2026-11-05T00:00:00Z`. A bare date `2026-11-05` means midnight UTC. The game uses
the player's device clock.

## Images

Upload a PNG to `images/` and set `banner` to
`https://sunsetclusterleadership.github.io/bushwack-news/images/<name>.png`. About 1600 x 229
(7:1); the News row centre-crops it to about 1.8:1, so keep the important part in the middle.
PNG only, under 4 MB. Until it downloads, the built-in banner for the post's kind shows.

## Tone

- Warm and short: **3 to 6 lines** of body.
- Talk to the player as a gardener and a friend; thank them.
- Sign off with two lines: `"Sincerely,"` then `"The Bushwack TD Team"` (an empty `""` line
  before the sign-off reads well).
- Spell the game **Bushwack TD** (capital B, capital TD, no hyphen).
- Bullets (`• `) for lists of changes; no walls of text.

## Validation, before every commit

1. The file parses as JSON (paste it into https://jsonlint.com, or `python -m json.tool feed.json`).
2. `"version"` is still `1`.
3. Every id is unique across posts and rewards, and has never been used before (check the Post
   history).
4. Every time is valid UTC (`YYYY-MM-DDTHH:MM:SSZ`) and every `date` is `YYYY-MM-DD`.
5. Commas: one between entries, none after the last one. Straight quotes `"`, not curly ones.
6. Optional, from the game repository: run the game's own validator, `MailboxFeed.parse_text` in
   `scripts/ui/mailbox/mailbox_feed.gd`, on the file and check it reports no problems and keeps
   every entry.
7. Commit to `main`, wait a minute or two, open the feed address in a browser and check the entry
   is there.

## Post history

| id | kind | date | notes |
|---|---|---|---|
| `mailbox-v2-update` | update | 2026-10-30 | bundled |
| `legacy-welcome` | notice | 2026-10-23 | bundled, hidden 2026-11-03 |
| `legacy-thanks-for-testing` | notice | 2026-10-23 | bundled, hidden 2026-11-03 |
| `welcome-testers-2026-11` | notice | 2026-11-03 | tester welcome |
