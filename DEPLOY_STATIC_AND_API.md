# Glass Ludo – Static Site + Web Service

| Part | What it holds | Where it runs |
|---|---|---|
| **Static site** (`static-site/`) | Graphics, animations, game logic, sounds, icons, PWA files | Any static host (Render Static, Netlify, Cloudflare Pages, GitHub Pages) |
| **Web service** (`ludo.py`, API mode) | Rooms, server-side dice, WebSocket + long-poll | Anything that runs Python (Render, Railway, Fly.io, VPS) |

The page finds the web service through ONE line in `static-site/config.js`:

    window.GLASS_API = "https://glass-ludo-api.onrender.com";

## What changed in the script
- `ludo.py --build-static <folder> --api <url>` writes the static site for you.
- `API_ONLY=1` makes `ludo.py` a pure web service (no game page).
- `ALLOWED_ORIGINS` limits which sites may call it (CORS + WebSocket origin check). Default `*`.
- The page reads `window.GLASS_API` (config.js); sounds, icons and manifest use relative paths.
- Invite links now point at the static site, not the API address.
- Plain `python ludo.py` still works exactly as before (everything in one).

## Deploy on Render (easiest, both parts)
1. Push this folder to GitHub.
2. Render > New > **Blueprint** > pick the repo (uses `render.yaml`): creates `glass-ludo-api` and `glass-ludo-site`.
3. Copy the API's URL (e.g. `https://glass-ludo-api-xxxx.onrender.com`). Render adds a suffix if the name was taken.
4. Put it in `static-site/config.js` (or run `python ludo.py --build-static static-site --api <url>`), commit, push.
5. In the API service > Environment set `ALLOWED_ORIGINS` = the static site URL (no trailing slash).
6. Open the static site URL, Play Online > Create room > Invite friends.

## Other hosts
- Static: upload the `static-site/` folder (Netlify drag-and-drop, Cloudflare Pages, GitHub Pages with `.nojekyll`).
- Web service: `API_ONLY=1 ALLOWED_ORIGINS=https://your-site PORT=8000 python3 ludo.py`
  - Must be reachable over **https/wss** when the site is https (browsers block http/ws from an https page).
  - Needs WebSocket support (Render, Railway, Fly.io all do).

## Test locally
    API_ONLY=1 PORT=8000 python ludo.py                 # terminal 1
    python ludo.py --build-static static-site --api http://localhost:8000
    cd static-site && python -m http.server 5000        # terminal 2 -> http://localhost:5000

## Notes
- Free Render web services sleep when idle (first open ~1 min); rooms live in memory and vanish on restart.
- After changing the game code, rebuild the static site (`--build-static`) and redeploy it.
- If the page says offline: check the URL in `config.js`, `ALLOWED_ORIGINS`, and open `<api-url>/health` (should say `ok`).
