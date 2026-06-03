# Deploying the SpikeWatch site to GitHub Pages

You already have **two GitHub repos** under `nischal-acharya27`:

| Existing repo                                  | What's in it                       |
| ---------------------------------------------- | ---------------------------------- |
| `nischal-acharya27/SpikeWatch` (private)       | Your app source code               |
| `nischal-acharya27/nischal-acharya27.github.io` (public) | Your personal site at `https://nischal-acharya27.github.io/` |

You want the SpikeWatch marketing site to live at
`https://nischal-acharya27.github.io/spikewatch/` without touching either
of those existing setups.

**Cleanest path: drop the marketing site into a `spikewatch/` subfolder
of your existing personal-site repo.** Your personal site keeps working
at the root, the SpikeWatch site lives at the `/spikewatch/` subpath, no
new repos, no conflicts.

| URL                                                | Content                            |
| -------------------------------------------------- | ---------------------------------- |
| `https://nischal-acharya27.github.io/`             | Personal site (unchanged)          |
| `https://nischal-acharya27.github.io/spikewatch/`  | This SpikeWatch marketing site     |

The private `SpikeWatch` app repo is untouched. The fact that GitHub repo
names are case-insensitive (which is why we can't make a separate repo
called `spikewatch`) doesn't matter here, because we're using a
*subfolder* — folders inside a repo can be any name.

---

## A note on privacy

If you'd ever wanted to publish Pages directly from the private
SpikeWatch repo, you'd need GitHub Pro / Team / Enterprise *and* the
site would still be publicly accessible to anyone with the URL — repo
privacy doesn't make the site private. True access-gated Pages only
exists on Enterprise. The setup below avoids all of that: SpikeWatch
source stays in its private repo; the marketing site lives in your
existing public personal-site repo.

---

## 0 · Security pre-flight (done)

A scan of every file in `site/` was clean: no usernames, email
addresses, absolute paths, API keys, tokens, or private keys. The
`site/.gitignore` excludes `.DS_Store`, the raw `data/` exports
(redundant — already inlined into `data_island.js`), and
`build_data_island.py`.

---

## 1 · Replace the placeholder email (one-time)

`hello@spikewatch.app` appears in four places:

| File                  | What it's used for                          |
| --------------------- | ------------------------------------------- |
| `index.html`          | Quant tier "Talk to us" mailto              |
| `index.html`          | Waitlist modal JS fallback                  |
| `legal/terms.html`    | Contact line                                |
| `legal/privacy.html`  | Privacy contact line                        |

Mass-replace from your terminal:

```sh
cd ~/Desktop/AntigravityProjects/SpikeWatch
grep -rn 'spikewatch\.app' site/                # confirm what's there
sed -i '' 's/cryptoassetsdaily@proton.me\.app/your.real@email.com/g' \
  site/index.html site/legal/terms.html site/legal/privacy.html
sed -i '' 's/cryptoassetsdaily@proton.me\.app/your.real@email.com/g' \
  site/legal/privacy.html
grep -rn 'spikewatch\.app' site/                # should now show nothing
```

The four URL meta tags (`canonical`, `og:url`, `og:image`,
`twitter:image`) already point to
`https://nischal-acharya27.github.io/spikewatch/`. The footer's GitHub
button was already swapped to a "View architecture →" link that
resolves on the live site, so anonymous visitors hit something real
even though the SpikeWatch source repo is private.

---

## 2 · Find your existing personal-site checkout (or clone it)

If you already have a local clone of your personal-site repo, skip to
step 3. Otherwise:

```sh
cd ~/Desktop/AntigravityProjects                     # or wherever you keep repos
git clone https://github.com/nischal-acharya27/nischal-acharya27.github.io.git
cd nischal-acharya27.github.io
ls                                                   # take a look so you know what's already there
```

You'll see whatever your personal site contains. **We're going to add a
new top-level folder named `spikewatch/` — nothing else changes.**

---

## 3 · Copy site/ in as the new spikewatch/ subfolder

From the personal-site repo root:

```sh
# You should be in your local clone of nischal-acharya27.github.io.
pwd                                                  # sanity check
git status                                           # confirm clean state

# Copy the marketing site contents into a NEW spikewatch/ subfolder.
# The trailing /. on the source copies dotfiles too, so the .gitignore
# I added travels with it.
mkdir -p spikewatch
cp -R ~/Desktop/AntigravityProjects/SpikeWatch/site/. ./spikewatch/

# Sanity check what landed.
ls -la spikewatch/                                   # see index.html, data_island.js, og.png, .gitignore, etc.
```

> **Important about the `.gitignore` inside `spikewatch/`.** It only
> applies to files inside that subfolder, so it won't affect anything
> in the rest of your personal-site repo. Good defaults: ignores
> `.DS_Store`, `spikewatch/data/`, and `build_data_island.py`.

Now commit + push:

```sh
git add spikewatch/
git status                                           # confirm only spikewatch/ files are staged
git commit -m "Add SpikeWatch marketing site at /spikewatch/"
git push
```

---

## 4 · Pages is already on for this repo

You already publish your personal site from this repo, which means
Pages is already configured. The new `spikewatch/` subfolder is served
automatically — no settings change needed.

GitHub typically rebuilds within a minute of the push. Watch
**<https://github.com/nischal-acharya27/nischal-acharya27.github.io/deployments>**
for the green checkmark.

---

## 5 · Verify the live site

Open `https://nischal-acharya27.github.io/spikewatch/` and check:

1. The hero candle animation runs and the dual-axis price/equity chart
   draws cleanly.
2. Nav "architecture →" loads
   `https://nischal-acharya27.github.io/spikewatch/architecture.html`
   (not a 404).
3. Footer **Risk disclosure / Terms / Privacy** links open the three
   stub pages.
4. Any "Get on the list" / "Get early access" button opens the waitlist
   modal.
5. DevTools → **Network** → reload → no 404s.
6. Paste the URL into <https://www.opengraph.xyz/> to preview how the
   +442.8% OG image renders on social cards.
7. Open `https://nischal-acharya27.github.io/` to confirm your **personal
   site is still there, untouched**.

If you get a 404 on `og.png` or `data_island.js`, open the GitHub web UI
for the personal-site repo and confirm those two files made it into the
`spikewatch/` folder (the `.gitignore` shouldn't swallow them, but worth
checking).

---

## 6 · Updating the site later

The simplest workflow — mirror `site/` from the private SpikeWatch
checkout into the personal-site repo's `spikewatch/` folder, then push:

```sh
# 1. Make changes in ~/Desktop/AntigravityProjects/SpikeWatch/site/
#    For a new backtest: Export Backtest from the app → drop in
#    site/data/ → run site/build_data_island.py to refresh data_island.js.

# 2. Mirror into the personal-site repo and push.
cd ~/Desktop/AntigravityProjects/nischal-acharya27.github.io
rsync -av --delete \
  --exclude=.git \
  --exclude=.gitignore \
  ~/Desktop/AntigravityProjects/SpikeWatch/site/ ./spikewatch/
git add spikewatch/
git commit -m "Refresh spikewatch site"
git push
```

The `--exclude` flags keep the spikewatch/ folder's `.gitignore`
intact while every other file mirrors `site/`. The trailing `/` on
both rsync paths matters — it tells rsync "copy contents into the
target", not "copy the source dir as a child of the target".

---

## 7 · Optional: auto-deploy via GitHub Actions

Instead of running rsync by hand, you can have a workflow in the
*private* SpikeWatch repo push the `site/` folder to the personal-site
repo's `spikewatch/` subfolder whenever `site/**` changes on `main`:

```yaml
# .github/workflows/publish-site.yml — IN THE private SpikeWatch repo
name: Publish site
on:
  push:
    branches: [main]
    paths: ['site/**']
jobs:
  push-to-public:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Mirror site/ to personal-site repo's /spikewatch/ subfolder
        env:
          PUBLIC_REPO_TOKEN: ${{ secrets.PUBLIC_REPO_TOKEN }}
        run: |
          git config --global user.name  "github-actions"
          git config --global user.email "actions@github.com"

          git clone \
            https://x-access-token:${PUBLIC_REPO_TOKEN}@github.com/nischal-acharya27/nischal-acharya27.github.io.git \
            /tmp/personal-site
          rm -rf /tmp/personal-site/spikewatch
          mkdir /tmp/personal-site/spikewatch
          cp -R site/. /tmp/personal-site/spikewatch/

          cd /tmp/personal-site
          git add spikewatch/
          if git diff --staged --quiet; then
            echo "No changes to publish."
          else
            git commit -m "Sync SpikeWatch site from $GITHUB_SHA"
            git push
          fi
```

Requires a fine-grained Personal Access Token with **Contents: write**
scope on `nischal-acharya27/nischal-acharya27.github.io`, stored as the
secret `PUBLIC_REPO_TOKEN` in the SpikeWatch repo's
**Settings → Secrets and variables → Actions**.

This is more cautious than a force-push: it merges into your
personal-site repo's history, so your personal-site commits stay
interleaved with the SpikeWatch refresh commits.

---

## 8 · Custom domain (optional, much later)

If you ever buy a domain (e.g. `spikewatch.app`) and want
*just the SpikeWatch site* on it — not your whole personal site — the
subfolder setup is the wrong topology for that. You'd need to break
SpikeWatch out into its own repo at that point. Not a problem now;
mentioned here so you don't get surprised later.

---

## 9 · Optional hardening

- **`security.txt`.** Drop a `.well-known/security.txt` in the
  personal-site repo root (not the `spikewatch/` subfolder — it should
  live at the apex):

  ```
  Contact: mailto:your.real@email.com
  Expires: 2027-01-01T00:00:00.000Z
  Preferred-Languages: en
  ```

- **CSP meta tag.** GitHub Pages doesn't let you set HTTP response
  headers, but you can ship one inside the `<head>` of
  `spikewatch/index.html`:

  ```html
  <meta http-equiv="Content-Security-Policy" content="
    default-src 'self';
    script-src 'self' 'unsafe-inline' https://cdn.tailwindcss.com;
    style-src 'self' 'unsafe-inline';
    img-src 'self' data:;
    font-src 'self' data:;
    connect-src 'self';
    frame-ancestors 'none';
    base-uri 'self';
    form-action 'self' mailto:;
  ">
  ```

  `'unsafe-inline'` weakens the policy; replacing the Tailwind play CDN
  with a self-hosted built CSS file lets you drop it.
