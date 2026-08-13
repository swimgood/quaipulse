# Chain Heartbeat

An ambient, generative audiovisual piece for Quai Network. A central pulsing core, expanding
light rings, and drifting sparks — each one triggered by a real, live transaction on Quai,
paired with a soft generative tone. Nothing here is simulated: if the chain goes quiet, so does
this. No backend, no build step, one HTML file.

## Deploy

**GitHub + Vercel dashboard**
```bash
git init
git add .
git commit -m "Chain Heartbeat"
git branch -M main
git remote add origin https://github.com/<you>/<repo>.git
git push -u origin main
```
Then import the repo at [vercel.com/new](https://vercel.com/new) — framework preset **Other**,
no build command needed.

**Vercel CLI**
```bash
npm i -g vercel
vercel
```

## How it works

- Polls `GET https://quaiscan.io/api/v2/transactions?filter=validated&items_count=100` every 3
  seconds, with cache-busting (`cache: 'no-store'` + a timestamp query param) since Cloudflare's
  edge can otherwise serve the same cached response across several polls in a row, making
  activity look bursty instead of continuous
- Diffs against transaction hashes already seen; anything new becomes a "pulse"
- **Ring size / spark count** scale (log-scaled) with the transaction's QUAI value, so whales
  make a bigger splash without blowing out the screen
- **Color** distinguishes ledgers — mint/teal for Quai (Type 0), amber for Qi (Type 2)
- **Audio** is generative: every pulse plays a soft tone constrained to a pentatonic-style scale
  (so it's always musically coherent no matter how randomly transactions arrive), pitched lower
  for bigger transactions, higher/airier for small or zero-value ones, panned randomly left/right
- If more than ~20 new transactions land in one poll (a busy moment), pulses are capped and
  staggered across the next few seconds rather than all firing at once — keeps it watchable
  instead of chaotic
- Click the ⓘ info control to see a live debug readout: transactions received on the last poll,
  how many were actually new, and a countdown to the next poll — useful for checking the feed is
  keeping up with real activity rather than guessing from the visual alone
- Same Cloudflare/CORS caveat as the wallet scanner applies here: this only works from a real
  deployed site, not from inside a claude.ai artifact

## URL options (for OBS / stream overlay use)

- `?transparent=1` — transparent background instead of dark, for use as an OBS Browser Source
  layered over other content. In OBS: Add Source → Browser → paste your deployed URL with this
  param, check "Shutdown source when not visible" **off** so it keeps polling in the background.
- `?mute=1` — starts muted (useful if you're overlaying it and don't want the built-in audio,
  e.g. because you're scoring it separately)
- `?minimal=1` — hides all UI chrome (status dot, controls, info panel) immediately, for a
  completely clean overlay

These can be combined, e.g. `?transparent=1&minimal=1`.

## Notes

- Audio requires a user gesture to start (browser autoplay policy) — that's what the click-to-
  begin gate is for. In OBS, Browser Sources don't get a user click, so audio may stay silent
  there; use `?mute=1` and handle sound separately if you need it in a stream.
- Click the ⓘ info control to toggle a small session stats readout (total pulses, rate/min).
  Off by default to keep the piece purely visual.
