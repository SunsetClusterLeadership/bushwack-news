# Bushwack TD mailbox feed

This repository IS the game's mailbox. The game downloads one file from it:

**https://sunsetclusterleadership.github.io/bushwack-news/feed.json**

Everything in this repository is public on purpose: it is shown to every player.
Changing `feed.json` never needs a new game build. Players see a change the next time the game
fetches (at launch, or when they open the mailbox at least 10 minutes after the last fetch).
GitHub Pages can also take a few minutes to publish a commit.

The full format reference lives in the game repository at `docs/ops/mailbox-feed.md`. The short
version is below.

## Add a news post (the News tab)

1. On github.com open `feed.json` and press the pencil (Edit).
2. Copy an existing entry inside `"posts": [ ... ]`, from its `{` to its `}`, and paste it after
   the last one. Put a comma between entries (and none after the last one).
3. Give it a **new `id`** (see Ids, below).
4. Fill in the fields:

| Field | Required | What it is |
|---|---|---|
| `id` | yes | Unique text, never reused. |
| `kind` | yes | `update`, `event` or `notice`. |
| `title` | yes | Row title and the opened post's title. |
| `date` | yes | `YYYY-MM-DD`, shown on the row. The list is newest first. |
| `start`, `end` | no | UTC times. Hidden before `start` and from `end` on. |
| `banner` | no | `update`, `event` or `notice` (art built into the game), or the full https address of a PNG in `images/` (see Images). |
| `headline` | no | Big text over the banner in the opened post. |
| `body` | yes | A list of lines. A line starting with `• ` is a bullet; `""` is a paragraph gap. |
| `gift` | no | A list of reward items (below). Adds a Claim row to the post. |
| `gift_caption` | no | Words next to the gift. Default "A gift for you!". |

5. Press **Commit changes** (to `main`).
6. After a minute or two, open the feed address above in a browser and check your post is there.

## Add a gift (the Rewards tab)

Same steps, inside `"rewards": [ ... ]`:

| Field | Required | What it is |
|---|---|---|
| `id` | yes | Unique text, never reused, and not the same as any post's id. |
| `title` | yes | Row title. |
| `reason` | no | The line under it. |
| `items` | yes | A list of reward items. |
| `start` | no | UTC time. Hidden before it. |
| `expires` | yes | UTC time. The row counts down to it. |

Example:

```json
{
	"id": "sorry-wave-12-freeze",
	"title": "Sorry for the bug!",
	"reason": "Wave 12 froze for some players. Thanks for your patience.",
	"items": [{"type": "gems", "amount": 250}],
	"expires": "2026-11-20T00:00:00Z"
}
```

### Reward items

- `{"type": "gems", "amount": 250}`
- `{"type": "petals", "amount": 1000}`
- `{"type": "key", "tier": "common", "amount": 1}` — tier is `common`, `rare`, `epic`,
  `legendary` or `spell`.

Amounts are whole numbers above 0. The game caps one item at 50,000 gems, 100,000 petals or 20
keys.

## Ids — never reuse one

Each player's "read" and "claimed" are remembered **by id, forever**. Reusing an old id means
players who claimed the old gift cannot claim the new one, and players who read the old post see
the new one as already read. Use a fresh, descriptive id every time, e.g.
`event-halloween-2026`, `gift-launch-week-2026-12`. Never delete-and-reuse.

To take a post down, either delete its entry or set its `end` to a time in the past. (The three
posts already here also ship inside the game; deleting them here does not hide them, setting an
`end` in the past does.)

## Dates and expiry

- Times are UTC, written `2026-11-05T00:00:00Z`. A bare date `2026-11-05` means midnight UTC.
- The game uses the player's device clock.
- A gift shows from `start` until `expires`. After it expires it shows only to players who
  claimed it (as "Claimed"), for one more day.

## Images

1. Upload a PNG into `images/` (Add file > Upload files).
2. Use its full address as the post's `banner`, e.g.
   `https://sunsetclusterleadership.github.io/bushwack-news/images/harvest_wide.png`.

Size: about **1600 x 229** (7:1). The opened post shows the whole wide image; the News row
centre-crops it to about 1.8:1, so keep the important part in the middle. PNG only, under 4 MB
(aim for a few hundred KB). Until the image downloads, the game shows the built-in banner for the
post's `kind`.

## Before you commit: check the JSON

A broken file is ignored by the game: players keep the last good feed, but nothing new shows.
Paste the whole file into https://jsonlint.com and make sure it says valid. Common mistakes: a
missing comma between entries, a comma after the last entry, curly quotes instead of `"`.

## Files

- `feed.json` — the feed (`"version": 1` must stay).
- `images/` — banner PNGs.
- `.nojekyll` — tells GitHub Pages to serve files as they are. Leave it.
