# GVG Genetics Site — working rules

## ALWAYS: update the footer date on every change

**Any time anything on this site is changed, update the footer "Last updated" line
to the current month and year before committing.**

```html
<p class="updated">Last updated: September 2026</p>
```

It lives at the bottom of the `<footer>` block in `index.html`. This applies to every
edit — horse records, images, badges, copy, anything. No exceptions.

---

## Repo

- Live at **gvg-genetics.com**, served by **GitHub Pages** from `GVG-GVG/gvg-genetics`, branch `main`.
- **Work directly in this folder. Never clone the repo.**
- Commit as: `git -c user.name="GVG" -c user.email="graham.gochneaur@gmail.com" commit`

### Never use `git add -A`

Stage explicit paths only. The working folder contains files that are not in the repo,
and `-A` stages their deletion. This has already nearly deleted:

- **`CNAME`** — contains `gvg-genetics.com`. Deleting it takes the custom domain offline.
- `pedigree-causeforcommotion.pdf` — still referenced by the site.

`.gitignore` covers `.DS_Store`, `BACKUP_index_*.html`, `backups/`, and local working docs.

### Push credentials

The PAT is **not** in the remote URL. It lives in `.git/.git-credentials` (mode 600,
inside `.git`, so it can never be committed), read via:

```
credential.helper = !git credential-store --file "<abs path>/.git/.git-credentials"
```

The `!` prefix and the quotes are required — the repo path contains a space, and an
unquoted `store --file=...` value makes git split the path and fail with
`could not read Username`.

**Before deleting old tokens on GitHub, check which one git is actually using.** A token
listed as "Never used" is not the one authenticating pushes. Deleting the in-use token
breaks `push` while `ls-remote` and `clone` keep working, because read access on a public
repo is anonymous — so the repo looks reachable right up until you try to write.

### Don't touch the DNS records that serve the site

Four A records (`185.199.108–111.153`) and `www` CNAME → `gvg-gvg.github.io`.
DNS is at GoDaddy. Mail records (MX/SPF/DKIM/DMARC) point to Google Workspace —
see "Email" below.

---

## Data model

`index.html` is a single file. Horses live in four JS arrays:
`racingHorses`, `broodmares`, `yearlings`, `foals`. Counts are derived from array
lengths — don't hardcode them.

Conventions:

- `raceHistory` is **ascending by age** (age 2, then 3, then 4…).
- `racingRecord` `{starts, wins, places, shows}` must equal the sum of the
  `raceHistory` rows. Verify before committing.
- `achievement` is a string or array. Earnings badges go **last** and read
  `Over $XXXK in Earnings`. Producer badges say `Produce Earnings Over $XXXK` —
  the distinction matters, since the usual phrasing means the horse's *own* race earnings.
- Badges containing "Champion" or matching `^G\d\s` render maroon; everything else gold.

## Images

- **Never crop off a horse's legs or head.** Downscale, don't crop.
- Profile images use the `image` field; previous mains move to the front of `gallery`.
- Naming: `{horse}-{year}-{month}.jpg`, e.g. `nubility-25-aug.jpg`.
- Convert large PNGs to JPG (quality ~88) — source files are often 5 MB+.

---

## Email

`gvg@gvg-genetics.com` on Google Workspace Business Starter (account index `/u/6/`).
The consumer @gmail.com accounts in the same browser are **not** the Workspace account.

The contact form posts to **FormSubmit** (`formsubmit.co`). Changing the destination
address requires clicking a confirmation link sent to the new address before
submissions will deliver.

Display the address **obfuscated** (assembled in JS), not as a plain `mailto:` in the HTML.

---

## Verifying work

Parse the arrays and check them rather than eyeballing:

```bash
node -e "
const fs=require('fs');
const s=fs.readFileSync('index.html','utf8');
const bm=eval(s.match(/const broodmares = \[[\s\S]*?\n\];/)[0].replace('const broodmares =',''));
// ...assert record totals, check referenced image files exist, etc.
"
```

Before committing image changes, confirm every `image` and `gallery` path resolves
to a file that actually exists on disk.
