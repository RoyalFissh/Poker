# Poker Ledger

A single-page app Kevin uses at his home game to track buy-ins, rebuys, cash-outs,
who owes who, chip denominations, seat draws, and per-player stats.

**Everything lives in `index.html`.** One file: HTML, CSS, and vanilla JS in an
IIFE. No build step, no framework, no package manager, no dependencies. Do not
add any — the whole point is that the file can be edited and pushed from a phone.

## Deploy

The repo *is* the site: GitHub Pages serves `main` at
<https://royalfissh.github.io/Poker/>. So:

1. Edit `index.html`.
2. Commit and push to `main`.
3. Wait ~40s and confirm the live bytes actually changed:

```bash
curl -s https://royalfissh.github.io/Poker/ | grep -c "some-string-you-just-added"
```

Verify by fetching the URL, not by the Pages API — `builds/latest` lags and
reports stale commits.

## How the code is organised

Inside the IIFE, in order: money/time helpers → persistence (`load`, `save`,
`normalizeDb`) → roster → type-ahead → Venmo → modal → chip grid → game math →
render functions per tab → graph → editors → `renderAll` → wiring.

- **State** is one object, `db`, persisted to `localStorage` under
  `poker-ledger:v2`. Every write goes through `save()`.
- **`normalizeDb()` is the safety net.** Any new field on `db` gets a default
  and a type guard there, so an old save or a hand-edited import can't crash the
  app. Add to it whenever you add state.
- **`renderAll()`** re-renders every tab. Inline updates (`updateNetInline`,
  `updatePlanTotals`, `updateClock`) exist so typing in a field doesn't blow away
  focus — use them for keystroke-level updates.
- **Money is computed in integer cents** (`chipsToDollars`) so 5¢ chips don't
  drift. `round2()` everything that reaches storage.
- **Always `escapeHtml()`** anything player-typed that goes into `innerHTML`.
- **Errors surface as a toast** via `window.onerror` — a blank screen means a
  syntax error, so check that first.

## House style Kevin has asked for

- **Dollars lead, chips are secondary.** Every money input defaults to a plain
  dollar amount; counting chips by colour is the opt-in path. In a
  dollars/chips toggle, `$` goes first and starts active.
- **Compact sizing.** Table / Chips / Seats / Pay carry `class="panel compact"`
  and are scaled to match the Stats tab's Player stats table (~13px body,
  ~10px labels). Stats is the reference — leave it alone. New markup in those
  panels needs a matching `.panel.compact` rule.
- `switchTab` toggles the `active` class with `classList` — never assign
  `className` on a panel, it drops `compact`.
- Inputs are under 16px, which is only safe because the viewport meta sets
  `maximum-scale=1` to stop iOS Safari zooming on focus. Keep that meta tag.
- **ROI reads `(+245.5%)`** — sign and percent both inside the brackets. It is
  colour-coded green/red (via `cls()`) everywhere it appears now, including
  next to a profit figure — Kevin decided he wanted it to match the money
  figures instead of staying neutral grey.
- **BB/hr reads `+$2.3 BB/hr`** — `fmtBB()` puts a `$` right after the sign,
  even though it's a big-blind count, not a dollar figure. Kevin asked for the
  `$` explicitly, for visual consistency with the other stats columns.
- **Player stats + Profit graph live in their own Stats tab**, not History.
  Tab order is Table, Chips, Seats, Stats, History, Pay. History is just Past
  games / Roster / Backup &amp; restore now. The Graph button on a stats row
  (`showPlayerGraph`) just scrolls to the Profit graph card within the same
  Stats tab — it doesn't switch tabs, since both live together now.
- Prefer small inline SVG over any charting library.

## Two gotchas that have cost real time

- **Pages sends `Cache-Control: max-age=600`.** Browsers honour it for ten
  minutes without even revalidating, and the `<meta http-equiv>` cache tags in
  the file do not override it. So "close and reopen the app" is not a reliable
  way for Kevin to pick up a fix. Once `curl` confirms the push is live, send
  him a cache-busted link — `https://royalfissh.github.io/Poker/?fix2` — any
  throwaway query string works, and his data survives because `localStorage` is
  scoped to the origin, not the query string.
- **iOS keeps a Safari tab's `localStorage` separate from a home-screen icon's**
  for the same URL. If Kevin says his data vanished, ask which one he opened
  before assuming a save bug. History → Backup & restore moves data between them.

## Testing

Kevin has no node or python installed. To check a change, open `index.html` in a
browser and drive it, or serve it with a PowerShell `HttpListener` script. Note
`localStorage` is disabled on `data:` URLs, so a file preview won't persist
state between reloads — rebuild test data with a script in the console.

## On the back burner

"Editing a game while it's running" — Kevin asked for it, then said he wasn't
sure what he meant. Ask him what it should do before building anything.
