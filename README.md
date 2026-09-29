# Meal Ledger

A one-tap diet tracker. No food logging, no macros, no calorie counting.
Each meal slot gets a single rating and the app does the arithmetic.

**Live:** https://pspgaspar.github.io/meal-ledger/

## The scale

Five levels, symmetric, centred on zero. A day's net above zero is a net
good day; a week is the sum of its days.

| Rating   | Points | Meaning            |
|----------|-------:|--------------------|
| Perfect  |     +2 | on target or under |
| Good     |     +1 | close enough       |
| Average  |      0 | breakeven          |
| Bad      |     −1 | over target        |
| Terrible |     −2 | full cheat meal    |

Unplanned extras cost −1 each, capped at −3 a day. Every label, point value
and description is editable in Setup at runtime.

## Where the data lives

`localStorage` is the working copy, so a tap is never blocked on the network.
A private GitHub Gist is the durable copy, written through a debounced push
after each change. Paste the same token on another device and everything
comes back.

The token needs the single `gist` scope and is stored only in the browser.
It is never committed here.

## Data model

- `settings` — slot targets, rating definitions, point values
- `days[YYYY-MM-DD]` — `{ ratings, extras, note, updatedAt }`

Days store the rating **name**, never a computed score. Scores are derived at
render time, so changing a point value in Setup recomputes every week already
logged.

Merges are per-day, newest `updatedAt` wins, so two devices that are both
behind cannot erase each other.

Weight tracking is intentionally not built. It drops in as its own
`weights[YYYY-MM-DD]` collection without touching any existing day.

## Files

| File | Role |
|------|------|
| `index.html` | the whole app — vanilla JS, no build, no dependencies |
| `sw.js` | service worker, network-first with an offline fallback |
| `manifest.webmanifest` | makes it installable to a phone home screen |

## Developing

There is no build step. Edit `index.html` and push; GitHub Pages serves it.
To check the JavaScript parses before pushing:

```sh
sed -n '/^<script>/,/^<\/script>/p' index.html | sed '1d;$d' > /tmp/ml.js
node --check /tmp/ml.js
```

## Automatic exercise from Apple Health

A web page cannot read Apple Health. HealthKit is native-only and has no browser
API, so this is a limit of iOS rather than of the app. Apple Shortcuts *can*
read it and can make an HTTP request, so a nightly automation fills in Exercise
with no backend and no App Store.

The Shortcut writes to `exercise-inbox.json`, a **separate file** in the same
Gist. It never touches `meal-ledger.json`, so an outside writer can add to the
exercise log and cannot damage ratings, weights or settings. Verified: adding a
file to a Gist leaves the other files byte-identical.

The app reads the inbox on every open and folds new figures into Exercise. A
value already applied is skipped, so a correction typed by hand survives the
automation running again; only a genuinely different reading overrides it.

### Setting it up

1. Create a second GitHub token with only the `gist` scope.
2. Shortcuts app, Automation tab, new Personal Automation, Time of Day, around
   23:45, Run Immediately.
3. Actions:
   - **Find Health Samples** — type Active Energy, filtered to today.
   - **Calculate Statistics** — Sum.
   - **Round Number** — to a whole number.
   - **Text**, containing exactly:

     ```
     {"files":{"exercise-inbox.json":{"content":"{\"DATE\":NUMBER}"}}}
     ```

     Replace `DATE` with a Current Date variable formatted `yyyy-MM-dd`, and
     `NUMBER` with the rounded sum. The backslashes are literal: the inner
     object is a JSON *string*, which is how the Gist API takes file content.
   - **Get Contents of URL**
     - URL `https://api.github.com/gists/<your gist id>`
     - Method `PATCH`
     - Headers: `Authorization: Bearer <token>`, `Accept: application/vnd.github+json`
     - Request Body: Text, set to the Text action above.

The Gist ID and the exact body shape are shown in the app under Setup, in
"Exercise from Apple Health", so they can be copied on the phone.

Action names vary a little between iOS versions, and this recipe has not been
run on a device.
