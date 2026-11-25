# enkitech.de

Static site for `enkitech.de`, a personal page under the name Enkitech (a personal brand,
not a registered company). Served by GitHub Pages. No build step, no third-party requests
(fonts are served from this site; no CDNs or analytics).

- `index.html`: the whole site on one page: name, role, a short note and contact on top,
  then the projects as rows (`#projects`, newest first; a comment at the end of the list
  says how to add the next one). Menu: Home (`/`), Notes (`/notes/`),
  theme switch. Decided 2026-11-25.
- `notes/`: my writing, a page of its own, called Notes (chosen 2026-11-25 so short
  pieces fit). "No notes yet." until the first one; a comment there shows the list markup.
- `projects/`: only a redirect to `/#projects`, so old links keep working.
- `privacy/`: privacy policy. There is no Impressum: the site is run as a personal,
  non-commercial page (no company, nothing sold, no offer of services), by the owner's
  decision on 2026-11-23, confirmed 2026-11-24. No street address or location on the site.
- `404.html`: page for unknown paths.
- `assets/site.css`: styles. Newsreader (headings) and Inter (text), self-hosted in
  `assets/fonts/` under the SIL Open Font License (licence files next to them). Light and
  dark mode follow the system setting until the visitor uses the switch in the header;
  `assets/theme.js` keeps that choice in the visitor's browser under `enkitech_theme`
  (mentioned on the privacy page). The
  frame is a full-height flex column, so the footer rests at the bottom on short pages.
- `assets/og.png`: the 1200×630 image shown when the site is shared (LinkedIn and others).
  It was rendered from an HTML page with headless Chrome; if the name or role changes,
  re-render it rather than editing the PNG.
- `assets/favicon.svg`, `favicon-32.png`, `apple-touch-icon.png`: site icons.
- `CNAME`: the custom domain GitHub Pages serves this repo on.

Pages link to `/assets/…` with root-relative paths (needed for `404.html`, which GitHub
serves at any depth), so preview from the repo root:

`python3 -m http.server 8812`, then open http://127.0.0.1:8812/.

Plain HTML on purpose (decided 2026-11-23). Switch to Hugo when the first blog post is
ready: same toolchain and Pages deploy workflow as `athyr-tech/athyr-website`, own layouts
from the current HTML and `site.css`, no third-party theme (keeps the no-external-requests
rule).

## Before publishing

All placeholders are filled in. If you add new ones, mark them `TODO-…`;
this must print nothing before you publish:

```bash
grep -rn 'TODO-' --include='*.html' .
```

Contact email is `contact@enkitech.de` (iCloud+ Custom Email Domain; MX, SPF and DKIM are
set at GoDaddy). The privacy text is a reasonable starting point, not legal advice.

## Hosting

Live since 2026-11-23. All three sites are GitHub Pages with HTTPS enforced; DNS for
`enkitech.de` is at GoDaddy.

| Address | Served from | Old address (still works) |
| --- | --- | --- |
| `enkitech.de` (`www` redirects here) | `kasfreedom/enkitech.de`, `main` / root | — |
| `sabura.enkitech.de` | `sabura-app/sabura-app.github.io`; the editor at `/sabura-editor/` comes from `sabura-app/sabura-editor` | `sabura-app.github.io/…` → 301 to the same path |
| `athyr.enkitech.de` | `athyr-tech/athyr-website` (Hugo, Actions deploy from `master`) | `athyr.tech` → see below |

### DNS records for enkitech.de

| Type | Name | Value | For |
| --- | --- | --- | --- |
| A | `@` | `185.199.108.153`, `.109.153`, `.110.153`, `.111.153` | GitHub Pages |
| AAAA | `@` | `2606:50c0:8000::153` … `8003::153` | GitHub Pages |
| CNAME | `www` | `kasfreedom.github.io.` | GitHub Pages |
| CNAME | `sabura` | `sabura-app.github.io.` | Sabura |
| CNAME | `athyr` | `athyr-tech.github.io.` | Athyr |
| MX, TXT (`apple-domain`, SPF), CNAME `sig1._domainkey` | | iCloud+ | `contact@enkitech.de` |

GoDaddy had a default forwarding rule to an Afternic "for sale" page, which locked the
parking A records; it was removed. Don't add wildcard (`*`) records.

### athyr.tech

`athyr-tech/athyr.tech` holds the `athyr.tech` domain (`www.athyr.tech` redirects to it):

- It serves `install.sh` as a file, so `curl -sSL https://athyr.tech/install.sh | sh` from
  old docs keeps working (`curl` doesn't follow HTML/JS redirects).
- Every other path shows a page that sends browsers to the same path on athyr.enkitech.de.

`install.sh` exists in four places and must stay identical: `scripts/install.sh` in the
private `athyr` repo (the source), `athyr-tech/athyr-bin`, `static/install.sh` in
`athyr-website`, and `athyr-tech/athyr.tech`. Keep `athyr.tech` registered while old links
may be around; if it lapsed, someone else could serve a script at that URL.

`athyr.tech` has no MX record, so it can't receive mail. Contact addresses in the Athyr
repos (`athyr-sdk-python` author, `mcpx` SECURITY.md, the brand guide) use
`contact@enkitech.de` instead.

### Things to know

- **Certificates:** GitHub didn't request a certificate for any of the three domains by
  itself, because each custom domain was set before its DNS pointed to GitHub. Fix: clear
  the custom domain, check it's empty, set it again (state goes to `new`, then `approved`
  within minutes), then enforce HTTPS:

  ```bash
  gh api -X PUT repos/OWNER/REPO/pages -F cname=null
  ```

  ```bash
  gh api -X PUT repos/OWNER/REPO/pages -f cname=DOMAIN
  ```

  ```bash
  gh api -X PUT repos/OWNER/REPO/pages -F https_enforced=true
  ```

- **Domain verification:** don't verify `enkitech.de` under the personal account. That
  would lock its subdomains to `kasfreedom`, and the `sabura-app` and `athyr-tech`
  organizations would lose theirs.
- **Sabura's in-board link** (top-bar icon in the editor and in every saved board) stays
  `https://sabura-app.github.io/` on purpose: saved boards keep it forever, and GitHub
  redirects it to whatever custom domain is current (see `src/ui/topbar.js`).
- **Check everything:**

  ```bash
  for u in https://enkitech.de/ https://sabura.enkitech.de/sabura-editor/ https://athyr.enkitech.de/docs/ https://athyr.tech/install.sh; do curl -sS -o /dev/null -w "%{http_code} $u\n" "$u"; done
  ```
