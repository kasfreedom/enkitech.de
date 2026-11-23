# enkitech.de

Static site for `enkitech.de`, a personal page (about me, my work, contact) under the
name Enkitech (a personal brand, not a registered company). Served by GitHub Pages.
No build step, no third-party requests (no web fonts, CDNs or analytics).

- `index.html`: home page: about me and contact.
- `projects/`: the projects (Sabura, Athyr; a commented template in the
  `<ul class="projects">` list for adding the next project).
- `privacy/`: privacy policy. There is no Impressum: the site is run as a personal,
  non-commercial page (no company, nothing sold), by the owner's decision on 2026-11-23.
- `assets/site.css`: styles: one quiet column, serif headings, system fonts only, light and
  dark mode. Deliberately plainer than the projects' own sites, so each project keeps its
  own look. `assets/theme.js`: light/dark switch (stored in the visitor's browser under
  `enkitech_theme`).
- `CNAME`: the custom domain GitHub Pages serves this repo on.

Preview locally: `python3 -m http.server 8812` and open http://127.0.0.1:8812/.

Plain HTML on purpose (decided 2026-11-23). Switch to Hugo when the first blog post is
ready: same toolchain and Pages deploy workflow as `athyr-tech/athyr-website`, own layouts
from the current HTML and `site.css`, no third-party theme (keeps the no-external-requests
rule).

## Before publishing

All placeholders are filled in. If you add new ones, mark them `TODO-…` in a
`<span class="todo">`; this must print nothing before you publish:

```bash
grep -rn 'TODO-' --include='*.html' .
```

Contact email is `contact@enkitech.de` (iCloud+ Custom Email Domain; MX, SPF and DKIM are
set at GoDaddy). The privacy text is a reasonable starting point, not legal advice.

## Hosting plan

| Address | Served from | Notes |
| --- | --- | --- |
| `enkitech.de` | GitHub repo `kasfreedom/enkitech.de` (this folder), Pages from `main` / root | A *project* repo, not `kasfreedom.github.io`, so other project pages of the personal account are not moved under enkitech.de |
| `www.enkitech.de` | same | GitHub redirects `www` to the apex automatically when both are set up |
| `sabura.enkitech.de` | `sabura-app/sabura-app.github.io` | Same repo, new address (was `sabura-app.github.io`); see below |
| `athyr.enkitech.de` | `athyr-tech/athyr-website` (Hugo, deployed by GitHub Actions) | Same repo, new address (was `athyr.tech`); see below |
| `athyr.tech` | keeps redirecting to `athyr.enkitech.de` | Must keep working: published install command is `curl -sSL https://athyr.tech/install.sh \| sh` |

Three GitHub accounts serve these: `kasfreedom` (apex), the `sabura-app` org and the
`athyr-tech` org. Each subdomain is a CNAME to its own account's `github.io` host, and each
repo sets its own custom domain. Nothing here needs a new website, only DNS records and
Pages settings, plus the URL changes listed in each move section.

### 1. Create the repo and turn on Pages (needs your go-ahead)

Only after the TODO grep above prints nothing: once pushed and published, the pages are
public at `kasfreedom.github.io/enkitech.de/` even before DNS points anywhere.

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
`sabura-app` and `athyr-tech` organizations can no longer serve `sabura.enkitech.de` and
`athyr.enkitech.de`. Since both are moving there: **don't verify the apex under your
personal account.** GitHub's docs don't say whether an organization can verify just a
subdomain; try it in each org's Settings → Pages and use the TXT name the UI shows.
Otherwise rely on steps 1 and 3 (domain set in Pages before DNS, no wildcard records).

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
| CNAME | `sabura` | `sabura-app.github.io.` |
| CNAME | `athyr` | `athyr-tech.github.io.` |

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

```bash
dig athyr.enkitech.de +noall +answer -t CNAME
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

## Serving Sabura at sabura.enkitech.de

The repo stays where it is; only the address it is served on changes.

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
- The "Visit the Sabura site" link in `projects/index.html` already points to
  `https://sabura.enkitech.de/`, so do this before or together with publishing enkitech.de.

## Serving Athyr at athyr.enkitech.de

The repo stays where it is; only the address it is served on changes.

Today `athyr-tech/athyr-website` is live at `https://athyr.tech/` (Pages `cname` is
`athyr.tech`, DNS has the four GitHub A records). A Pages site serves exactly one custom
domain, so switching it means `athyr.tech` stops being served by that repo.

**Plan the `athyr.tech` redirect before switching, or `curl -sSL https://athyr.tech/install.sh | sh`
breaks** for everyone who copied it from the docs, the blog post or the README. `curl -L`
follows HTTP redirects but not HTML/JS redirects, so a redirect page alone isn't enough.
(The hero on the home page shows `curl -sSL athyr.tech/install | sh`, without `.sh`; that
path already returns 404 today. Change the hero to `athyr.enkitech.de/install.sh` in step 3.)

1. **Optional but recommended:** verify `athyr.tech` in the `athyr-tech` org (org Settings →
   Pages → Add a domain, TXT record at the registrar). Only that org uses the domain, so
   there's no catch, and nobody else can claim it during the switch.
2. Choose how `athyr.tech` keeps answering:
   - **Your DNS provider's URL forwarding** (simplest, if it offers it): a permanent (301),
     path-preserving forward of `athyr.tech` and `www.athyr.tech` to
     `https://athyr.enkitech.de`. Check it serves HTTPS, otherwise `https://athyr.tech/…`
     fails with a certificate error. Set it up at step 5, replacing the GitHub A records.
   - **A small redirect repo** in the `athyr-tech` org (for example `athyr-tech/athyr.tech`):
     a copy of `install.sh` served as is (so `curl` still gets the script), and an
     `index.html` and `404.html` that send browsers on to the same path at
     `https://athyr.enkitech.de`. Create it now with Pages on but **no custom domain yet**:
     GitHub doesn't let two repos hold `athyr.tech`, so it can only take the domain after
     step 5. Keep the A records as they are. Downside: two copies of `install.sh` to keep in
     step.
3. In `athyr-website`, prepare (don't push yet): `baseURL = 'https://athyr.enkitech.de/'`
   in `hugo.toml`, `static/CNAME` → `athyr.enkitech.de`, and replace `athyr.tech` in
   `content/docs/installation.md`, `content/blog/introducing-athyr.md`,
   `layouts/index.html` and `static/install.sh` (usage comment and docs link).
4. Add the `athyr` CNAME record (table above) so the new name resolves.
5. Switch, all in one go:
   - `gh api -X PUT repos/athyr-tech/athyr-website/pages -f cname=athyr.enkitech.de`
   - Point `athyr.tech` at the redirect: either set up the forwarding, or
     `gh api -X PUT repos/athyr-tech/athyr.tech/pages -f cname=athyr.tech`.
   - Push the step 3 changes. The deploy workflow takes its base URL from
     `actions/configure-pages`, which reads the Pages domain, so the cname must be changed
     before this build (otherwise re-run the workflow). With an Actions deploy GitHub
     ignores the `CNAME` file and uses the setting, but keep the file in step anyway.

   Expect a short gap (minutes, up to an hour) while GitHub issues certificates for the
   new names; HTTPS on `athyr.tech` may fail during it. Do it at a quiet time.
6. Once the certificates are there, enforce HTTPS:
   `gh api -X PUT repos/athyr-tech/athyr-website/pages -F https_enforced=true`, and the
   same for `athyr-tech/athyr.tech` if you used the redirect repo.
7. Check: `curl -sSL https://athyr.tech/install.sh | head -3` prints the script,
   `curl -sSL https://athyr.enkitech.de/install.sh | head -3` too, and
   https://athyr.enkitech.de/docs/ and https://athyr.tech/docs/ both land on the docs.
8. Update other places that say `athyr.tech`: READMEs in the `athyr-tech` repos (`athyr`,
   `athyr-sdk-go`, `athyr-sdk-python`, `homebrew-tap`, …), the GitHub org profile and
   repo "website" fields. (The "Visit the Athyr site" link in `projects/index.html` already points to
   `https://athyr.enkitech.de/`.)

Keep `athyr.tech` registered for as long as old links and install commands may be out
there; letting it lapse would let someone else serve a script at that URL.
