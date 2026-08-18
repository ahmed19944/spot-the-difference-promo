# spot-the-difference-promo

The cross-promotion feed for the **Spot The Difference** series. One JSON file that every shipped
issue of the game reads on launch to find out which issue is current, so a build from month one can
still point at the issue released in month six.

**Live feed:** https://ahmed19944.github.io/spot-the-difference-promo/issues.json

There is no build step and no server. GitHub Pages serves these files as they are; editing
`issues.json` here in the browser is a complete release.

The game side lives in [ahmed19944/spot-the-difference](https://github.com/ahmed19944/spot-the-difference)
— `scripts/promo.gd` and `Docs/CrossPromotion.md`.

---

## Adding an issue — the whole job

1. Drop the cover into `covers/`, named in lowercase with dashes: `covers/veil-of-embers.jpg`.
2. Open `issues.json` and add an entry to the top of the `issues` list. Copy the shape from
   `_example`, which is there for exactly this and is ignored by the game.
3. Commit. Pages redeploys in under a minute and every installed build picks it up on its next
   launch.

**Newest first.** The game shows the first entry in the list that is not the build doing the looking,
so ordering is the whole of the routing: issues one through five all land on six, and six lands on
whatever you add after it. Nothing is per-issue and nothing needs maintaining.

**Leave old entries in place.** They cost nothing, and they are what a future issue points back at if
you ever reorder.

---

## The entry

```json
{
  "id": "VeilOfEmbers",
  "title": "Veil of Embers",
  "blurb": "Ten new scenes",
  "cover": "https://ahmed19944.github.io/spot-the-difference-promo/covers/veil-of-embers.jpg",
  "urls": {
    "steam": "https://store.steampowered.com/app/000000/VeilOfEmbers/",
    "android": "https://play.google.com/store/apps/details?id=com.std.veilofembers",
    "ios": "https://apps.apple.com/app/id000000"
  }
}
```

| Key | Required | Notes |
| --- | --- | --- |
| `id` | yes | **The theme folder name**, exactly as it appears under `assets/Themes/` in the game repo, and as `theme_name` in that issue's `theme_config.tres`. Getting this wrong is the one mistake with a visible symptom: a build will advertise *itself*. |
| `title` | yes | As it reads on the store page. It wraps to two lines in the card at roughly 18 characters. |
| `blurb` | no | One short line. Leave it out and the card closes the gap rather than showing an empty row. |
| `cover` | yes | Absolute URL. PNG, JPEG or WebP — the game sniffs the format from the file's own bytes, so the extension does not have to be honest. |
| `urls` | yes | At least one. Desktop opens `steam`, mobile opens `android` or `ios`, and each falls back to any sibling that is present rather than to nothing. |

Unknown keys are ignored, so you can leave notes to yourself anywhere in the file.

## Verifying a build without touching the live feed

`test/issues.json` is a fixture serving one card over the same real HTTPS the shipped game uses. Its
`id` is `__TestIssue__`, which no theme folder will ever be called, so every build shows it.

```
-- --promo-feed=https://ahmed19944.github.io/spot-the-difference-promo/test/issues.json
```

A debug build launched with that should draw the card in the menu's bottom-left corner within a
second or two of the title appearing. If it does, the whole path works: TLS, Pages, the JSON, the
cover download and the cache. Then point it at the real feed and the only remaining variable is the
contents of `issues.json`.

## Cover art

- Any aspect — the card centres it and preserves the ratio — but **square-ish reads best**, because
  the card is a 4:3 plate with the art on the left.
- Longest edge around **512 px**, JPEG quality 85, comfortably **under 200 KB**.
- Anything below **32 px** or above **2048 px** on an edge is rejected. Both are valid images that
  would wreck the card's layout, so the game refuses them rather than drawing them.

## If something is wrong

Every failure is silent by design: the game keeps whatever it last showed, or shows nothing, and the
menu looks untouched. It never draws a half-finished card. So a broken feed does not break anything —
it just quietly stops working, which means it is worth checking rather than waiting to be told.

Fastest check — the feed must be reachable and parse:

```sh
curl -s https://ahmed19944.github.io/spot-the-difference-promo/issues.json | python -m json.tool
```

Then confirm each `cover` URL returns `200`:

```sh
curl -s -o /dev/null -w "%{http_code}\n" <cover-url>
```

Common causes, in the order they happen: a trailing comma in the JSON; a `cover` URL pointing at a
file that was never committed; an `id` that does not match the theme folder; or an `http://` URL,
which works everywhere except iOS and so fails on the one platform hardest to test.

> **URLs must be `https://`.** iOS App Transport Security rejects cleartext.
