# **Hobobiker Rides Again**

## Part 3: The Long Haul

Rebuilding the site in Drupal 11

![DDEV Logo](images/ddev-logo.svg)

Note:
Same site, same test suite, a very different target: a working Drupal 11 site with the content, URLs and access rules of the original.

---

## Last week

* The static archive is live: [static.hobobiker.com](https://static.hobobiker.com)
  (GitHub Pages, [rfay/hobobiker-static](https://github.com/rfay/hobobiker-static))
* **11,679** checks pass, 0 fail
* But not on the first try...

Note:
Final local run of retire/run.sh, every tier, against the nginx-served archive (DISCOVERIES.md, "Part 2 re-run after the cleanup"). GitHub Pages serves it at the custom domain from rfay/hobobiker-static. The next two slides are what happened between that first green run and this one.

---

## Confession 1: I changed the original

* The maps were broken: ancient, dead Google Maps key
* **434** day maps, **13** route maps
* Fixed on production: Leaflet + OpenStreetMap, no key required
* Baseline re-frozen to match

Note:
"Reproduce, don't repair" is the rule for the migration, but I couldn't bear it. The Google key belonged to an unknown project with no billing, so every map had been blank for years. 13 route maps had a second bug: a leftover debugging line put https://hobobiker.com/ in front of an absolute URL, so the KML never loaded. The repair is in the tripinfo module: each map is a div with a site-relative data-kml URL, drawn by Leaflet 1.9.4 with togeojson and JSZip, all served locally. Because the original changed on purpose, the old baseline no longer described it: re-frozen 2026-10-25, the old one stays in git history.

---

## Confession 2: secrets in public

* Personal data in 3 map files: names, addresses, phone numbers
* An AWS key, a dead Google key, SMTP and Flickr credentials
* GitHub whined mightily about the Google Maps Key when it was in hundreds of pages of static.
* Published in the archive repo and the data release
* Revoked, deleted, and started over

Note:
The personal data: WarmShowers hosts and family contacts in XML files under files/maps, about 75 phone numbers. The credentials were in the variable table and, the worst one, an AWS access key inside a Backup and Migrate destination URL. drush sql-sanitize only scrubs users, not these. All of it was public in rfay/hobobiker-static and in hobobiker-assets v1.0 and v1.1. What it took: rotate the AWS key and delete the old one, delete the Flickr app and every credential variable, delete both GitHub repos, remove the files from production. Then a sanitized snapshot with a gate that searches every table and file for keys, a re-frozen baseline, a secrets gate in the static pipeline, and the whole archive rebuilt, retested (11,679 pass, 0 fail) and redeployed. The data is now hobobiker-assets v1.2. Lesson: the audit has to look at every table, not just users and variables.

---

## Confession 3: I didn't design the tests

Somebody asked last week "What if you just told Claude to do it and gave no guidance?"

Part 1's prompt asked for a plan and a way to test it, but I didn't figure out the tests

* Claude proposed the tiers and built them
* I sort-of reviewed them; I never specified them

Note:
Last week's question: "What if you just asked Claude to do this without the tests?" It would do something, and say it was done. Without a check, "done" is the agent's word; with the baseline, it is a count anyone can rerun. Part 1's recap on drupal.org is linked on the last slide.

---

## DDEV v1.25.5: upgrade

* PHP security fixes, among other things
* `ddev import-db` and `ddev export-db` handle **`.zst`**
* So from now on we use `.zst` dumps

Note:
Everyone should upgrade. Zstandard dumps are smaller and much faster to compress and decompress than gzip; hobobiker-assets v1.2 ships both db.sql.gz and db.sql.zst. The workspace for this series runs v1.25.5.

---

## The target

`hobobiker-d11`: a fresh Drupal 11, nothing in it yet

* Drupal 11.4, PHP 8.4, MariaDB 11.8
* A sibling DDEV project, like `hobobiker-static`
* The same suite, with `MIGRATION_TARGET=drupal11`

Note:
Fresh install from the recommended project template. The original, hobobiker-copy, stays the reference; the D6 database to read from is a copy of the sanitized snapshot, loaded into a second schema (d6) of the target's database, never the original.

---

## The bar is higher than for static

* Private pages answer **403**, not 404
* Stored content counts: **447 comments**, 82 terms
* Every file, byte for byte
* The `static:` tier no longer applies

Note:
Last week's static-only decisions (404 for private pages, the removed search block) are scoped to the static target, so they don't excuse anything here. The stored tier compares the database, not the pages: comments never render on the original (module off), but a CMS rebuild that drops them is a loss. The kit now has a Drupal 11 exporter (scripts/stored-exporters/drupal11.mjs, kit 7db0c15), tested with a broken copy: one comment deleted and one title changed in the Drupal 11 database, both named.

---

## What has to come over

* **1,296** nodes, 12 types (1,255 published)
* **1,601** published paths, every node ID kept
* **1,793** files, 1.1 GB
* 447 comments, 82 terms, users

Note:
From the frozen baseline (data/manifest.json, baseline/stored.json, re-frozen 2026-10-25). Types: triplog 549, blog 342, wp 138, story 113, acidfree 77, page 35, book 18, and a few small ones. Keeping node IDs keeps /node/N links in old content working; aliases come from url_alias.

---

## Two ways to move content

* **Migrate API**: core's `d6_*` plugins, YAML, rollback
* **A rerunnable script**: read the D6 copy, write entities, keep IDs

Both should work. We'll see what Claude does.

Note:
The kit's docs/rebuild-approaches.md. Migrate API suits a standard Drupal 6 source and long, staged migrations. A script suits one cutover and odd content; randyfay.com went that way (126 nodes, 670 comments, 33 files in about 8 seconds). No 6-to-7-to-8 upgrade chain either way. Which one we use comes out of the plan. Rehearsed with scripts, as on randyfay.com: the original ids kept, so node/N and every alias still work.

---

## The pipeline

1. **Restore** the D6 copy and a clean D11 (D6 database loaded)
2. **Configure**: types, fields, formats, from git
3. **Import** content, comments, terms, files
4. **Check** against the baseline

Every step is a script.

Note:
The source here is a copy of the original's database in a second schema of the target's database. Reading the source directly works too, and is common: DDEV projects share a network, so the target's web container reaches the original with mysql -h hobobiker-copy-db (ddev mysql -h does not). Same shape as Part 2: always from a clean restore, never by hand-editing the target, so the run that goes green is reproducible. Configuration lives in git (config export or a recipe). Rehearsed: a fresh install and the Drupal 6 copy reloaded from .zst in 21 s, the whole pipeline in 38 s. Content types stay as on the original (triplog, blog...) rather than collapsing into one, as the audit suggests for a Drupal target.

---

## Looks

The tests compare words, images and links and try for "shape" of page.

1. Stock theme, plus a documented allowlist
2. A subtheme with the old CSS
3. Try to validate D11 database content if done from script.

Note:
The kit's docs/theming-and-presentation.md. Run the semantic tier with no theme entries and count failures by category; that is what the theme costs. On randyfay.com a Bartik subtheme (about 100 lines of CSS, 7 templates) took the allowlist from 8 entries to 3. Rehearsed: Olivero passed every word-and-link check and looked nothing like the site. The original theme was only about 880 lines (golden_hour plus hobo_golden), so it was ported: a stable9 theme whose templates produce the original's markup, so its CSS, images and menu script work unchanged. The layout tier then caught one thing a person might not: the trip day pages relied on Drupal 6 core's book.css. Visible and layout tiers: 4,905 of 4,905.

---

## Decisions

Every failure is one of:

* A gap in the baseline or the suite: fix it
* A real loss: fix the import
* A deliberate change: declare it, `target: drupal11`

Note:
Same rule as Part 2, with expected-differences.json entries scoped to the drupal11 target, exact (equals or anchored regex), each with a reason.

---

## Today

1. Onboard Claude: read the D6 source, write the plan
2. Build the pipeline, step by step
3. Run the suite; read the failures
4. Decide: fix, or declare

Note:
Pre-baked checkpoints for the long steps (file copy, a full import). Whatever passes or fails at the end is the result. Tag: s3-d11-run. If time runs out, the rehearsed pipeline (hobobiker-d11 main, 2026-10-28): reset 21 s, pipeline 38 s, every tier 4 min 18 s, 10,669 pass, 0 fail (semantic 2,457, visible 2,455, layout 2,450, assets 1,781, stored 1,445, access 81), 10 declared differences: an empty title and a comment on a deleted node (Drupal 11 can't store them), the rewritten PHP lines, /tripinfo (a PHP fatal error on the original), one view argument, an image link now an image style, two self-links where a path has two aliases. Found on the way: the original printed dates at a fixed UTC-6 but filtered by America/Denver; three MP3 sizes capped at 8 MB in its files table; ties broken by table order.

---

## The opening prompt

What a fresh Claude gets today:

* The owner's decisions, as requirements
* What to read, then explore the D6 source
* First deliverable: a plan, not code

[PART3-PROMPT.md](https://github.com/rfay/hobobiker.com/blob/main/PART3-PROMPT.md)

Note:
Pasted into a new Claude Code session in hobobiker-d11 on the live-start branch, after scripts/00-reset.sh. The decisions are as I gave them, not the rehearsal's solutions; it is told not to look at the rehearsal (other branches, tags, PART3-PLAN.md), and the rehearsal's drupal11 declarations are hidden for the live build.

---

## Followup: what the tests missed

* Click a photo: the original opens a lightbox, Drupal 11 opened the bare image
  * Same markup, no script. The tests don't click.
* 168 comments shown that the original never showed (its comment module was off)
  * Extra text passes: the tests check that the original's lines are there
* Pager shown as a bullet list; edit forms with no body
  * No test looks at styling or logs in
* 30 term descriptions with no text format: found by redoing a step with the Migrate API
* Migrate skipped a term and reported "0 failed"
* Comparing both sites page by page (text, breadcrumbs, screenshots): 11 more
  * Breadcrumbs, titles, field order, line breaks from D6's HTML corrector

Found by a person, or by a second method.

Note:
Live build, 2026-10-30. The lightbox: node/1335 on the original runs the Drupal 6 Lightbox2 module's lightbox.js on every rel="lightbox" link; the hobophoto filter on Drupal 11 printed the same rel="lightbox[group1][caption]" links, so every word, image and link check passed, but no script was loaded. Fixed by loading the original lightbox.js unchanged with a small adapter for Drupal 11 (jQuery 4 in noConflict mode, no $.browser, drupalSettings, behaviors as {attach}) and porting its caption route. Comments: the original stores 447 (169 published) with the comment module disabled, so it shows none; the import kept 446 and Drupal 11 displayed the 168 published ones. Now kept in the database (the stored tier checks them) and hidden: no comment field on the node display, no "access comments" permission. Pager: Drupal 11's nav.pager markup is not styled by the original's CSS; the theme now prints D6's theme_pager() list, identical on all 193 baseline pages with a pager, and D6's handling of a page past the end (Views: page 1; the blog module: the last page). Edit forms: the build set up how nodes display but no edit forms, so node/N/edit showed only the base fields. Favicon: Drupal 11 linked core's icon. Term descriptions: setDescription() ignores a format argument; neither the tests nor validate() noticed, the Migrate version of step 30 did, and step 99 now checks for it. Migrate: term 31's parent has a higher tid, the parent's stub had no vocabulary, and the row was skipped with only a notice in the message table. Page-by-page comparison (2026-10-31): extracted text, breadcrumbs, titles, feed links and pagers on every baseline page, plus full-page screenshots of 123 pages compared pixel by pixel. It found the submitted line on poll and other types, case-insensitive tag order, breadcrumbs from D6's router paths and hidden navigation links, the site name and slogan in the title, trip tables that left out the end date, CCK fields placed after the body by weight and label, and D6's HTML corrector, which turned each "<br/>" into an extra "</br/>" line break in 69 bodies (ported exactly). Two deliberate changes, declared: a teaser's unclosed comment is closed (D6 hid four posts after it), and the Flash audio player is HTML5. After the fixes: 77 of 123 screenshots identical, 122 within 0.5%.

---

## Resources

* Archive: [static.hobobiker.com](https://static.hobobiker.com), from [rfay/hobobiker-static](https://github.com/rfay/hobobiker-static) · data: [hobobiker-assets v1.2](https://github.com/rfay/hobobiker-assets/releases/tag/v1.2)
* Recaps on drupal.org: [Part 1](https://www.drupal.org/docs/develop/development-tools/ai-coding-tools-for-drupal-development/drupal-ai-learners-club-sessions/2026-10-16-%E2%80%94-part-1-hobobiker-rides-again-plan-it-test-it-migrate-it-%E2%80%94-road-test-having-claude) · [Part 2](https://www.drupal.org/docs/develop/development-tools/ai-coding-tools-for-drupal-development/drupal-ai-learners-club-sessions/2026-10-23-%E2%80%94-part-2-hobobiker-rides-again-plan-it-test-it-migrate-it-%E2%80%94-the-last-ride-sending-a) · [Part 3](https://www.drupal.org/docs/develop/development-tools/ai-coding-tools-for-drupal-development/drupal-ai-learners-club-sessions/2026-10-30-%E2%80%94-part-3-hobobiker-rides-again-plan-it-test-it-migrate-it-%E2%80%94-the-long-haul-planning-and)
* Live during the session: [original](https://hobobiker-copy--remotework--rfay.coder.ddev.com) · [Drupal 11](https://hobobiker-d11--remotework--rfay.coder.ddev.com) · [Drupal11 from rehearsal](https://hobobiker-d11-rehearsed--remotework--rfay.coder.ddev.com)
* [Drupal migrations from Drupal 6/7](https://www.drupal.org/docs/upgrading-drupal)
* Site source: [github.com/rfay/hobobiker.com](https://github.com/rfay/hobobiker.com)
* The kit: [github.com/rfay/site-migration-kit](https://github.com/rfay/site-migration-kit)
* [ddev.com](https://ddev.com)

Note:
Live sites go away after the session; the archive and the assets release stay.
