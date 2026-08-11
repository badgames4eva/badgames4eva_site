# badgames4eva.com — publisher landing page

The apex-domain site: one page listing the games, plus the two ad-authorization files
that have to live at a domain root. Deployed by **Cloudflare** from this repo's `main`
branch via Cloudflare's Git integration — **pushing to `main` is the deploy.**

Live: <https://badgames4eva.com/>

## Why this repo is separate from the games

Two things genuinely require the *apex* domain, so a redirect from a game subdomain
wouldn't have worked:

1. **AdSense verifies the registrable domain** (`badgames4eva.com`) and its crawler wants
   the `google-adsense-account` meta tag served from that exact host.
2. **`app-ads.txt` must be at the root** of the developer website named in the app-store
   listing — per the IAB spec, not a subdomain and not a path.

And a landing page is the right thing to put there anyway: each game lives on its own
`gamename.badgames4eva.com`, and this page is the index over them.

## AdSense on this page

`index.html`'s `<head>` carries **two** AdSense things, and they do different jobs — keep
both:

1. `<meta name="google-adsense-account" content="ca-pub-5597688543726963">` — proves
   ownership of the domain to AdSense's crawler.
2. The `adsbygoogle.js` loader `<script>` — what actually lets Google place Auto ads.
   Google's instruction is to put it in `<head>` on **every page**; this site has one page,
   so that's this one. Add it to any page you add.

Keep `async` and `crossorigin="anonymous"` exactly as Google gives them. Neither the
publisher ID nor the script tag is a secret — it's public markup by design.

## Files

| File | What it is |
|---|---|
| `index.html` | The whole site. One self-contained file: no stylesheet, no build step, no script of our own — same house style as the game repos. It does load Google's AdSense script (see below). |
| `app-ads.txt` | Authorized sellers for the **apps**. Per-publisher, so one file covers every game, forever. |
| `ads.txt` | Authorized sellers for **web** pages on this host. |
| `.gitignore` | Copied from `words_on_demand` so the two repos behave the same. |

There is deliberately **no `CNAME` file**. That's a GitHub Pages mechanism; Cloudflare
ignores it and takes its custom domains from the dashboard instead. (The `words_on_demand`
repo still has one for historical reasons — it's inert there too, now that Cloudflare
serves that domain.)

## Cloudflare setup

Connected via **Workers & Pages → Create → Connect to Git**. The deploy serves the repo's
files as static assets straight from `main` — no framework, no build command, output at
the repo root. (You can tell it's Cloudflare's static-asset serving and not GitHub Pages:
a 404 returns a **zero-byte** body and every response carries `server: cloudflare`, where
GitHub Pages returns a styled HTML 404.)

The custom domains — `badgames4eva.com` and `www.badgames4eva.com` — are attached in the
Cloudflare dashboard. Because the zone is in the same Cloudflare account, Cloudflare
creates the proxied DNS records itself (flattened at the apex) and issues the certificate:
no manual A/AAAA records, and TLS terminates at the edge.

### The `www` → apex redirect

`www.badgames4eva.com` is a **301 redirect** to the apex, done with a Cloudflare **Single
Redirect** rule (Rules → Redirect Rules), *not* a second origin:

| Field | Value |
|---|---|
| Mode | Wildcard pattern |
| Request URL | `https://www.*` |
| Target URL | `https://${1}` |
| Status | 301 |
| Preserve query string | ✔ |

The `https://` scheme in the target is load-bearing: a scheme-less target like
`badgames4eva.com` is read as a *relative path*, leaving the browser on `www` (which has
no origin behind it) and producing a **522**. Verify from outside any browser cache:

```bash
curl -sI https://www.badgames4eva.com/app-ads.txt?x=1 | grep -iE '^(HTTP|location)'
# HTTP/2 301 ; location: https://badgames4eva.com/app-ads.txt?x=1
```

## Games listed here

| Game | Subdomain | Privacy page |
|---|---|---|
| Words on Demand | `wordsondemand.badgames4eva.com` | ✔ `/privacy` |
| Solitaire on Demand | `solitaireondemand.badgames4eva.com` | ✘ **none yet** — not linked rather than link a 404 |

Solitaire needs a privacy page before it goes to either store (both require a reachable
policy URL) — it stores game state and stats locally, so it has something to disclose.
Add the link to `.game-links` once it exists.

## Adding a game

1. Copy one `<a class="game">` block in `index.html`; edit the name, blurb, meta line, and
   `href`.
2. Add its privacy link to the `.game-links` paragraph below the grid.
3. Push.

The tile grid is `repeat(auto-fit, minmax(17rem, 1fr))`, so it reflows on its own as games
are added — no breakpoint to maintain.

Each game keeps its own repo on its own subdomain, so a game update never touches this
page. **`app-ads.txt` needs no change per game** — it authorizes the publisher, not the app.

## Store-listing consequence, easy to get wrong

Enter **`https://badgames4eva.com`** as the developer / publisher website in both store
consoles. `app-ads.txt` is looked for at the root of whatever is entered there, so naming a
game subdomain means it's never found and programmatic demand suffers. The
*privacy-policy* field is a different field and stays
`https://wordsondemand.badgames4eva.com/privacy`.

## Verify after a change

```bash
curl -sI https://badgames4eva.com/ | head -1                   # 200
curl -s https://badgames4eva.com/ | grep -c google-adsense      # 1 (ownership meta tag)
curl -s https://badgames4eva.com/ | grep -c adsbygoogle.js      # 1 (Auto ads loader)
curl -s https://badgames4eva.com/app-ads.txt | grep -v '^#'     # the DIRECT line
curl -sI https://www.badgames4eva.com/ | head -1                # 200 too
```

Locally, without touching the network:

```bash
python3 -m http.server 8899   # then open http://localhost:8899/
```
