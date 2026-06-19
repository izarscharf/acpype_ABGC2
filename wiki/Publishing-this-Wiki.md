# Publishing this Wiki

These pages live in the repository under [`wiki/`](../wiki) so they are version-
controlled and reviewed alongside the code. GitHub Wikis, however, are served from a
**separate** git repository (`<repo>.wiki.git`). To make these pages show up under the
repo's **Wiki** tab, publish them there.

## One-time setup

1. On GitHub, open the repo → **Wiki** tab → create the first page (e.g. click *Create
   the first page* and save). This initialises the `*.wiki.git` repo so it can be cloned.

## Publish / update

```bash
# clone the wiki repo (note the .wiki.git suffix)
git clone https://github.com/izarscharf/acpype_ABGC2.wiki.git

# copy the pages from the main repo into it
cp /path/to/acpype_ABGC2/wiki/*.md acpype_ABGC2.wiki/

cd acpype_ABGC2.wiki
git add .
git commit -m "Update ACPYPE (ABCG2 fork) wiki"
git push
```

GitHub renders:
- **`Home.md`** as the wiki landing page,
- **`_Sidebar.md`** as the navigation sidebar on every page,
- other `*.md` files as pages, linked by their filename without extension (e.g.
  `[Installation](Installation)` → `Installation.md`).

## Page naming & links

- Internal links use the page name without `.md` and **without** a directory, e.g.
  `[Charge Methods](Charge-Methods)`. These work once the files sit at the top level of
  the wiki repo.
- Links to repository files (e.g. `../acpype/topol.py`, `../HPC_SETUP.md`,
  `../environment.yml`) are written relative to the `wiki/` folder so they resolve when
  browsing the pages **in the main repo**. On the published GitHub Wiki those relative
  paths point outside the wiki repo — if you want them clickable there, replace them with
  absolute `https://github.com/izarscharf/acpype_ABGC2/blob/master/...` URLs during the
  copy step.

## Keeping them in sync

Treat [`wiki/`](../wiki) in the main repo as the **source of truth**, edit there, and
re-run the publish step above. A small CI action or a `make wiki` target can automate the
copy+push if you publish often.
