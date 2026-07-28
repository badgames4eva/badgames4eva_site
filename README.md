# badgames4eva.com — publisher landing page

The apex-domain site: one page listing the games, plus the two ad-authorization files
that have to live at a domain root. Deployed by **Cloudflare Pages** from this repo's
`main` branch — **pushing to `main` is the deploy.**

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

## Files

| File | What it is |
|---|---|
| `index.html` | The whole site. One self-contained file: no stylesheet, no script, no build step — same house style as the game repos. |
| `app-ads.txt` | Authorized sellers for the **apps**. Per-publisher, so one file covers every game, forever. |
| `ads.txt` | Authorized sellers for **web** pages on this host. |
| `.gitignore` | Copied from `words_on_demand` so the two repos behave the same. |

There is deliberately **no `CNAME` file**. That's a GitHub Pages mechanism; Cloudflare
Pages ignores it and takes its custom domains from the project settings instead. (The
`words_on_demand` repo still has one for historical reasons — it's inert there too.)

## Cloudflare Pages setup

Connected via **Workers & Pages → Create → Pages → Connect to Git**. Build settings, since
this is plain static files:

| Setting | Value |
|---|---|
| Framework preset | **None** |
| Build command | *(empty)* |
| Build output directory | `/` |
| Root directory | `/` |

Then **Custom domains → Set up a domain**, once for `badgames4eva.com` and once for
`www.badgames4eva.com`. Because the zone is in the same Cloudflare account, Cloudflare
creates the DNS record itself (a proxied record, flattened at the apex) and issues the
certificate — no manual A/AAAA records to add, and no grey-cloud dance, because Cloudflare
Pages terminates TLS at the edge by design.

Unlike GitHub Pages, a Cloudflare Pages project accepts **several** custom domains, so the
apex and `www` both attach to this one project.

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
curl -s https://badgames4eva.com/ | grep -c google-adsense      # 1
curl -s https://badgames4eva.com/app-ads.txt | grep -v '^#'     # the DIRECT line
curl -sI https://www.badgames4eva.com/ | head -1                # 200 too
```

Locally, without touching the network:

```bash
python3 -m http.server 8899   # then open http://localhost:8899/
```
