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

- Polls `GET https://quaiscan.io/api/v2/transactions?filter=validated&items_count=100` roughly
  every 2 seconds (with small random jitter, to avoid a bot-like exact cadence), with
  cache-busting since Cloudflare's edge can otherwise serve the same cached response across
  several polls in a row
- New transactions are queued, then dispensed with **adaptive pacing**: each item's delay is
  calculated from how much time is actually left until the next poll, so a batch of 4 spreads
  across the full ~2s gap instead of firing in under a second and then sitting idle
- **Experimental real-time layer**: also attempts a direct WebSocket connection to Quaiscan's
  Phoenix Channels endpoint (the same real-time mechanism Quaiscan's own `/txs` page almost
  certainly uses to update instantly). This is a best-effort guess at the channel path and
  payload shape — I haven't been able to verify Quaiscan's exact protocol from outside a
  browser. If it connects and joins successfully, the status line shows **"live (realtime)"**
  and updates arrive instantly instead of on a poll cycle. If it fails for any reason, it fails
  completely silently and the proven polling path keeps running exactly as before — there's no
  degraded or broken state, just no extra speed boost.
- **Color** — neon red for Quai (Type 0), amber for Qi (Type 2)
- Each burst spawns at a random point on screen, sized (log-scaled) by transaction value, with
  audio panned to match its position
- Click the ⓘ info control for a live debug readout (poll success/failure counts, last HTTP
  error, lifetime totals) and links to Quaiscan and Qu.ai
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
