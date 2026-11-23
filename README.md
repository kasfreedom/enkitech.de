# enkitech.de

Static site for `enkitech.de`, the personal home of the software published under the
name Enkitech (a personal brand, not a registered company). Served by GitHub Pages.
No build step, no third-party requests (no web fonts, CDNs or analytics).

- `index.html`: who's behind it, the software (Sabura; a commented template in the
  `<ul class="projects">` list for adding the next project), contact.
- `impressum/`: Impressum (German legal notice, § 5 DDG). `privacy/`: privacy policy.
- `assets/site.css`: styles, in the Sabura site's paper look. `assets/theme.js`: light/dark
  switch (stored in the visitor's browser under `enkitech_theme`).
- `assets/sketch/*.svg`: hand-drawn outlines copied unchanged from the Sabura site
  (`sabura-app/sabura-app.github.io`, `assets/sketch/`), which generates them from the
  Sabura editor's sketch code. To refresh them, copy them again from there.
- `CNAME`: the custom domain GitHub Pages serves this repo on.

Preview locally: `python3 -m http.server 8812` and open http://127.0.0.1:8812/.

## Before going live: fill in the placeholders

Placeholders are visible on the page, highlighted and marked `TODO-…`. **Do not publish
the site while any are left** — a `.de` site without a complete Impressum can be
warned off (abgemahnt). This must print nothing:

```bash
grep -rn "TODO-" --include=*.html .
```

| Placeholder | Where | What |
| --- | --- | --- |
| `TODO-NAME` | index, impressum, privacy | Your full name as it should appear |
| `TODO-STRASSE-HAUSNUMMER`, `TODO-PLZ-ORT` | impressum | A postal address where you can be reached (a "ladungsfähige Anschrift"; a P.O. box isn't enough) |
| `TODO-CONTACT-EMAIL` | index, impressum, privacy | A contact email address |

After replacing them, remove the `.todo` spans (keep the text). The Impressum and privacy
texts are a reasonable starting point, not legal advice.

## Hosting plan

| Address | Served from | Notes |
| --- | --- | --- |
| `enkitech.de` | GitHub repo `kasfreedom/enkitech.de` (this folder), Pages from `main` / root | A *project* repo, not `kasfreedom.github.io`, so other project pages of the personal account are not moved under enkitech.de |
| `www.enkitech.de` | same | GitHub redirects `www` to the apex automatically when both are set up |
| `sabura.enkitech.de` | `sabura-app/sabura-app.github.io` | Optional, separate step; see below |

### 1. Create the repo and turn on Pages (needs your go-ahead)

```bash
gh repo create kasfreedom/enkitech.de --public --source=. --push
```

```bash
gh api -X POST repos/kasfreedom/enkitech.de/pages -f 'source[branch]=main' -f 'source[path]=/'
```

```bash
gh api -X PUT repos/kasfreedom/enkitech.de/pages -f cname=enkitech.de
```

GitHub says to add the custom domain in Pages **before** creating the DNS records, so
nobody else can claim the domain on GitHub in between. Better still, verify the domain
first (next step), but read its catch.

### 2. Verify the domain with GitHub (optional, read the catch)

Verifying a domain on GitHub stops anyone else from publishing a Pages site on it after a
DNS slip. GitHub → your avatar → Settings → Pages → **Add a domain** shows a TXT record:

| Type | Name | Value |
| --- | --- | --- |
| TXT | `_github-pages-challenge-kasfreedom` | *(value shown by GitHub)* |

**The catch:** a verified domain *and its immediate subdomains* can then only be used by
repos of the account that verified it. If `kasfreedom` verifies `enkitech.de`, the
`sabura-app` organization can no longer serve `sabura.enkitech.de`. So:

- If Sabura will move to `sabura.enkitech.de`: don't verify the apex under your personal
  account. Instead let the `sabura-app` org verify `sabura.enkitech.de`
  (org Settings → Pages; TXT `_github-pages-challenge-sabura-app.sabura`), and rely on
  steps 1 and 3 (domain set in Pages before DNS, no wildcard records) for the apex.
- If Sabura stays at `sabura-app.github.io`: verify `enkitech.de` under your account.

These TXT records are **only for GitHub** and are separate from the DNS TXT record
Anthropic asks for to verify the domain for the Claude plugin Directory. TXT records don't
conflict; you can have all of them.

### 3. DNS records to add at your DNS provider

From GitHub's [Managing a custom domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)
(checked 2026-11-23). Remove any other A/AAAA records on `@` first (for example a
registrar's parking page) and any record for `www`; add no wildcard (`*`) records.

| Type | Name (host) | Value |
| --- | --- | --- |
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |
| CNAME | `www` | `kasfreedom.github.io.` |
| CNAME | `sabura` | `sabura-app.github.io.` *(only when you do the Sabura step below)* |

The CNAME targets are the account hosts, without any repository name. Some providers want
the trailing dot, some add it themselves. Email records (MX, SPF) are not affected.

### 4. Check DNS and HTTPS

```bash
dig enkitech.de +noall +answer -t A
```

```bash
dig enkitech.de +noall +answer -t AAAA
```

```bash
dig www.enkitech.de +noall +answer -t CNAME
```

```bash
dig sabura.enkitech.de +noall +answer -t CNAME
```

You should see the four `185.199.10x.153` addresses, the four `2606:50c0:800x::153`
addresses, and the CNAMEs above (DNS changes can take up to a day to show). Then:

```bash
gh api repos/kasfreedom/enkitech.de/pages --jq '{cname, status, https_enforced, protected_domain_state}'
```

Once GitHub has issued the certificate (minutes to an hour after DNS is right), enforce
HTTPS:

```bash
gh api -X PUT repos/kasfreedom/enkitech.de/pages -F https_enforced=true
```

Finally open https://enkitech.de/ and https://www.enkitech.de/ (should redirect to the apex).

## Moving the Sabura site to sabura.enkitech.de (separate decision)

Change in `sabura-app/sabura-app.github.io`:

1. Add a file `CNAME` containing `sabura.enkitech.de`, commit and push.
2. Set the custom domain in Pages:
   `gh api -X PUT repos/sabura-app/sabura-app.github.io/pages -f cname=sabura.enkitech.de`
3. Add the `sabura` CNAME record above, wait for the certificate, enforce HTTPS.

What that changes — read before deciding:

- **The editor moves too.** A custom domain on an organization's `*.github.io` site applies
  to every project site of that organization. `https://sabura-app.github.io/sabura-editor/`
  becomes `https://sabura.enkitech.de/sabura-editor/` (old URLs redirect).
- **Browser storage doesn't follow.** The new address is a different origin, so anything
  the hosted editor or site keeps in visitors' browsers (theme choice, autosaved boards in
  local storage, if any) is not visible at the new address. Check what the editor stores
  before moving, and tell users if they need to save boards as files first.
- Links and published references (plugin listing, README badges, docs) to
  `sabura-app.github.io` keep working through redirects, but should be updated.
- After the move, change the "Visit the Sabura site" link in `index.html` here to
  `https://sabura.enkitech.de/`.
