# **Hobobiker Rides Again**

## Part 3: The Long Haul

Rebuilding the site in Drupal 11

![DDEV Logo](images/ddev-logo.svg)

Note:
Same site, same test suite, a very different target: a working Drupal 11 site with the content, URLs and access rules of the original.

---

## Last week

* The static archive is live: [static.hobobiker.com](https://static.hobobiker.com)
* **11,679** checks pass, 0 fail
* Found on the way: credentials and personal data in published copies

Note:
Final local run of retire/run.sh, every tier, against the nginx-served archive (DISCOVERIES.md, "Part 2 re-run after the cleanup"). GitHub Pages serves it at the custom domain from rfay/hobobiker-static. The secrets: an AWS key in a Backup and Migrate destination, a Google Maps key, and personal data in three map XML files. drush sql-sanitize only scrubs users. All rotated or removed; the maps now use Leaflet. The pipeline now has a secrets gate, and the sanitized database and files are hobobiker-assets v1.2.

---

## The target

`hobobiker-d11`: a fresh Drupal 11, nothing in it yet

* Drupal 11.4, PHP 8.4, MariaDB 11.8
* A sibling DDEV project, like `hobobiker-static`
* The same suite, with `MIGRATION_TARGET=drupal11`

Note:
Fresh install from the recommended project template. The original, hobobiker-copy, stays the reference; the D6 database to read from is a copy (hobobiker-prep, restored from the pristine snapshot), never the original.

---

## The bar is higher than for static

* Private pages answer **403**, not 404
* Stored content counts: **447 comments**, 82 terms
* Every file, byte for byte
* The `static:` tier no longer applies

Note:
Last week's static-only decisions (404 for private pages, the removed search block) are scoped to the static target, so they don't excuse anything here. The stored tier compares the database, not the pages: comments never render on the original (module off), but a CMS rebuild that drops them is a loss. It needs a Drupal 11 exporter; the kit has only drupal6.mjs so far.

---

## What has to come over

* **1,296** nodes, 12 types (1,255 published)
* **1,601** published paths, every node ID kept
* **1,793** files, 1.1 GB
* 447 comments, 82 terms, users

Note:
From the frozen baseline (data/manifest.json, baseline/stored.json, re-frozen 2026-10-25). Types: triplog 549, blog 342, wp 138, story 113, acidfree 77, page 35, book 18, and a few small ones. Keeping node IDs keeps /node/N links in old content working; aliases come from url_alias.

---

## The hard parts

Content made at render time, not stored:

* **15** pages run PHP
* **15** route tables from `tripinfo`
* **119** pages with `[hobophoto:]` macros
* **13** Flash pages, broken already

Note:
Counts are the baseline's tags. The site is frozen, so one-time answers are fine: flatten the tripinfo tables to HTML once (the audit's recommendation), and either recreate a small hobophoto filter or expand the macros to markup. Flash stays as broken as it is on the original: reproduce, don't repair. The baseline already holds the rendered output of all of these, so the suite tells us whether the answer worked.

---

## Two ways to move content

* **Migrate API**: core's `d6_*` plugins, YAML, rollback
* **A rerunnable script**: read the D6 copy, write entities, keep IDs

Either is fine: the tests judge the result, not the method

Note:
The kit's docs/rebuild-approaches.md. Migrate API suits a standard Drupal 6 source and long, staged migrations. A script suits one cutover and odd content; randyfay.com went that way (126 nodes, 670 comments, 33 files in about 8 seconds). No 6-to-7-to-8 upgrade chain either way. Which one we use comes out of the plan.

---

## The pipeline

1. **Restore** the D6 copy and a clean D11
2. **Configure**: types, fields, formats, from git
3. **Import** content, comments, terms, files
4. **Check** against the baseline

Every step a script, every script prints a count

Note:
Same shape as Part 2: always from a clean restore, never by hand-editing the target, so the run that goes green is reproducible. Configuration lives in git (config export or a recipe). Content types stay as on the original (triplog, blog...) rather than collapsing into one, as the audit suggests for a Drupal target.

---

## Looks

The tests compare words, images and links; looks are a budget

1. Stock theme, plus a documented allowlist
2. A subtheme with the old CSS
3. A full port: rarely worth it

Note:
The kit's docs/theming-and-presentation.md. Run the semantic tier with no theme entries and count failures by category; that is what the theme costs. On randyfay.com a Bartik subtheme (about 100 lines of CSS, 7 templates) took the allowlist from 8 entries to 3. Today: start at step 1, go further only if time allows.

---

## Decisions

Every failure is one of:

* A gap in the baseline or the suite: fix it
* A real loss: fix the import
* A deliberate change: declare it, `target: drupal11`

Note:
Same rule as Part 2, with expected-differences.json entries scoped to the drupal11 target, exact (equals or anchored regex), each with a reason. An allowlist can hide real errors: on randyfay.com an entry that allowed every date line hid a 7-hour time zone mistake.

---

## Today

1. Onboard Claude: read the D6 source, write the plan
2. Build the pipeline, step by step
3. Run the suite; read the failures
4. Decide: fix, or declare

Note:
Pre-baked checkpoints for the long steps (file copy, a full import). Whatever passes or fails at the end is the result. Tag: s3-d11-run.

---

## Resources

* Archive: [static.hobobiker.com](https://static.hobobiker.com) · data: [hobobiker-assets v1.2](https://github.com/rfay/hobobiker-assets/releases/tag/v1.2)
* Live during the session: [original](https://hobobiker-copy--remotework--rfay.coder.ddev.com) · [Drupal 11](https://hobobiker-d11--remotework--rfay.coder.ddev.com)
* [Drupal migrations from Drupal 6/7](https://www.drupal.org/docs/upgrading-drupal)
* Site source: [github.com/rfay/hobobiker.com](https://github.com/rfay/hobobiker.com)
* The kit: [github.com/rfay/site-migration-kit](https://github.com/rfay/site-migration-kit)
* [ddev.com](https://ddev.com)

Note:
Live sites go away after the session; the archive and the assets release stay.
