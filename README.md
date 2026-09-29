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
