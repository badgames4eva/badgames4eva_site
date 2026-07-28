# badgames4eva.com — publisher landing page

The apex-domain site: one page listing the games, plus the two ad-authorization files
that have to live at a domain root. Deployed to GitHub Pages; **pushing to `main` is the
deploy.**

Live: <https://badgames4eva.com/>

## Why this repo exists separately from the games

GitHub Pages allows **one custom domain per repository**. `words_on_demand` already spends
its on `wordsondemand.badgames4eva.com` — an address that is baked into the app's
store-listing privacy URL and hard-coded in five files, so it can't move. The apex
therefore needs a repo of its own, and this is it.

Two things genuinely require the apex, so a redirect wouldn't have done:

1. **AdSense verifies the registrable domain** (`badgames4eva.com`) and its crawler wants
   the `google-adsense-account` meta tag served from that exact host.
2. **`app-ads.txt` must be at the root** of the developer website named in the app store
   listing — per the IAB spec, not a subdomain and not a path.

## Files

| File | What it is |
|---|---|
| `index.html` | The whole site. One self-contained file: no stylesheet, no script, no build step. |
| `CNAME` | Pins the Pages custom domain to `badgames4eva.com`. Deleting it silently reverts the site to `*.github.io`. |
| `app-ads.txt` | Authorized sellers for the **apps**. Per-publisher, so one file covers every game. |
| `ads.txt` | Authorized sellers for **web** pages on this host. |

## Adding a game

1. Copy one `<a class="game">` block in `index.html` and edit the name, blurb, meta line,
   and `href`.
2. Add its privacy link to the `.game-links` paragraph below the grid.
3. Push.

Each game keeps its own repo on its own subdomain (`gamename.badgames4eva.com`). This page
only holds names and links, so shipping a game update never touches it.

`app-ads.txt` needs **no** change per game — it authorizes the publisher, not the app.

## DNS

The zone is on Cloudflare (registrar and nameservers both). For Pages to serve the apex,
the DNS records must be:

| Type | Name | Value | Proxy |
|---|---|---|---|
| A | `@` | `185.199.108.153` | DNS only |
| A | `@` | `185.199.109.153` | DNS only |
| A | `@` | `185.199.110.153` | DNS only |
| A | `@` | `185.199.111.153` | DNS only |
| CNAME | `www` | `badgames4eva.github.io` | DNS only |

**Set the apex records to "DNS only" (grey cloud) until GitHub has issued the TLS
certificate.** Cloudflare's proxy terminates TLS itself, which hides the domain from
GitHub's Let's Encrypt challenge and leaves *Enforce HTTPS* stuck greyed-out in the Pages
settings. Once the cert is issued you can turn the proxy back on if you want Cloudflare's
caching.

Those four IPs are GitHub's published apex addresses; re-check them against
[GitHub's docs](https://docs.github.com/pages/configuring-a-custom-domain-for-your-github-pages-site)
if the apex ever stops resolving.

## Verify after a change

```bash
curl -sI https://badgames4eva.com/ | head -1                  # 200
curl -s https://badgames4eva.com/ | grep -c google-adsense     # 1
curl -s https://badgames4eva.com/app-ads.txt | grep -v '^#'    # the DIRECT line
```
