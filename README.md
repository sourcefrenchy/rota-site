# rota-site

The public marketing and support site for **ROTA — Watch Collection**.

Plain static HTML and CSS. No build step, no framework, no JavaScript, no web fonts,
no CDN, no analytics. An app whose entire pitch is "no analytics" cannot ship a site
that phones home, and a third-party font request *is* phoning home. Adding a tag here
would falsify the privacy page one directory over.

```
docs/
  index.html          Landing
  privacy/index.html  Privacy policy  ← required by App Store Connect
  terms/index.html    Terms of use / EULA  ← required for auto-renewing subscriptions
  support/index.html  Support  ← required by App Store Connect
  press/index.html    Press kit
  404.html            Not-found page (absolute links — see note below)
  assets/rota.css     The only stylesheet
  .nojekyll           Serve the tree verbatim; do not run Jekyll
  robots.txt
```

## The URLs, which must match three places byte-for-byte

Served from `/docs` on GitHub Pages, on a **public** repo named `rota-site` (Pages on a
private repo needs a paid plan; a separate public repo is free and cleanly isolates the
code):

| Page | URL |
|---|---|
| Landing | `https://sourcefrenchy.github.io/rota-site/` |
| **Privacy** | `https://sourcefrenchy.github.io/rota-site/privacy/` |
| **Terms** | `https://sourcefrenchy.github.io/rota-site/terms/` |
| Support | `https://sourcefrenchy.github.io/rota-site/support/` |
| Press kit | `https://sourcefrenchy.github.io/rota-site/press/` |

The two bold URLs appear in **three** places and all three must agree exactly, trailing
slash included:

1. `Packages/RotaFeatures/Sources/RotaFeaturePaywall/PaywallLegal.swift` (in the app repo)
   — already set to exactly these strings.
2. App Store Connect → App Information → **Privacy Policy URL**.
3. App Store Connect → the subscription group / each auto-renewing product →
   **Terms of Use (EULA)**.

Do not rename the directories. A directory rename changes a URL that is compiled into
a shipped binary.

## Before you publish — three blocking fill-ins

1. **`docs/terms/index.html`** contains a red "Before this page goes live" block and three
   placeholders in monospace: `[licensor legal name]`, `[licensor postal address]` and
   `[governing jurisdiction]`. Fill all three and delete the block.
   `grep -n '\[licensor\|\[governing' docs/terms/index.html` finds them.
2. **`docs/privacy/index.html`** has one placeholder in the Contact section for the
   controller name and postal address.
3. **The `rota.app` domain must actually be yours**, with working mailboxes for
   `support@`, `privacy@`, `legal@`, `press@` and `corrections@`. `corrections@rota.app`
   is a compiled-in constant in the app
   (`Packages/RotaFeatures/Sources/RotaFeatureWatchPage/Logic/CorrectionReport.swift`) and
   `support@` is the address App Store Connect will list. If the domain is not yours,
   change the addresses here **and** in that Swift file in the same commit — a support URL
   whose email bounces is a 1.5 rejection.

## Publishing

```bash
# 1 · Create the public repo (once).
gh repo create sourcefrenchy/rota-site --public \
  --description "Marketing and support site for ROTA — Watch Collection"

# 2 · Populate it from this directory's contents.
cd /tmp && rm -rf rota-site && git clone git@github.com:sourcefrenchy/rota-site.git
cp -R /path/to/Rota/Site/docs /path/to/Rota/Site/README.md /tmp/rota-site/
cd /tmp/rota-site
git add -A
git commit -m "Site: landing, privacy, terms, support, press kit"
git push

# 3 · Turn Pages on (once): Settings → Pages → Source: "Deploy from a branch",
#     Branch: main, Folder: /docs. Or:
gh api -X POST repos/sourcefrenchy/rota-site/pages \
  -f 'source[branch]=main' -f 'source[path]=/docs'
```

First deploy takes a couple of minutes. Then verify, before touching App Store Connect:

```bash
for p in "" privacy/ terms/ support/ press/; do
  printf '%-10s %s\n' "$p" \
    "$(curl -sSo /dev/null -w '%{http_code}' https://sourcefrenchy.github.io/rota-site/$p)"
done
```

All five must return `200`. App Store Connect fetches these URLs and rejects a version
whose privacy policy URL does not resolve.

Note the app repo is private and this one is public: never `git remote add` one to the
other, and never copy anything from `Proposal/` into this tree.

## Editing

Open a file, edit it, commit. There is nothing to install and nothing to compile.
The header and footer are duplicated across the five pages by design — five copies of
twelve lines is cheaper than a build step, and it is the only thing a build step would
have bought.

Two conventions worth keeping:

- **Relative links only** inside `docs/`, so the tree works identically at
  `/rota-site/`, at a custom domain, and from a local `python3 -m http.server`.
  `404.html` is the sole exception: GitHub Pages serves it for arbitrary missing paths,
  so its links are absolute and hard-code `/rota-site/`. If a custom domain is ever
  added, change `/rota-site/` to `/` in those five links.
- **Never** write that data does not leave the device. The privacy page's whole value is
  that it is checkable. `RotaFeatureSettingsTests` enforces the same rule inside the app,
  against `PrivacyManifesto.forbiddenPhrasings`; keep the site consistent with it.

## Checking a change

```bash
cd docs && python3 -m http.server 8000   # then open http://localhost:8000/
```

The site is 200-odd lines of CSS and five pages. If something looks wrong, it is in
`assets/rota.css`, and it is legible.
