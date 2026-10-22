# **Hobobiker Rides Again**

## Part 2: The Last Ride

Retiring a site to static HTML

![DDEV Logo](images/ddev-logo.svg)

Note:
Today we turn hobobiker.com into a folder of plain HTML files, and use last week's suite to prove nothing was lost.

---

## Last week

* A frozen baseline: 2,345 pages
* **10,613** checks pass on the original
* 7 of 7 deliberate breaks caught

Note:
Tag s1-baseline-frozen. The suite does not know or care how today's target is built. That is the point.

---

## Credit

**Karen Stevenson**, Lullabot

*Sending a Drupal Site Into Retirement* (three parts, 2020)

1. The approach
2. Using HTTrack
3. Using the Static Generator module

Note:
First published in 2014, republished as a series in 2020, all about Drupal 7 and GitHub Pages. Most of today's plan comes from it. Where we do something differently it is because of this site or this hosting, and any mistakes in applying it are ours. Links on the last slide.

---

## When to retire a site

* Nobody will edit it again
* Past events, old versions, a finished trip
* Drupal 6: no security updates since 2016

Note:
From the first article: expired event sites, rarely updated sites, sites whose next version starts clean. hobobiker.com is a finished bicycle trip. The content matters; the CMS no longer does.

---

> "One of the biggest problems of transforming a dynamic site into static pages is that the URLs must change."

— Karen Stevenson

Note:
/news becomes /news.html or /news/index.html. Our checks treat /a/b, /a/b/, /a/b.html and /a/b/index.html as the same page, and the static server's nginx rule answers the extensionless URL, so old links keep working.

---

## Prepare before you crawl

* Forms, search, login blocks: remove
* Comments: close
* Views: no AJAX, no exposed filters, fewer pagers
* Shortlinks and junk `<head>` links: remove
* Review as an anonymous visitor

Note:
The series' checklist, condensed. Everything a static site can't do goes before the crawler sees it. Shortlinks alone cut the article's HTTrack run by more than half.

---

## Survey first, then choose

Don't apply the generic list

Note:
Lesson from the randyfay.com rehearsal: the checklist reduced to two steps there, and "disable comments" would have deleted 670 real comments. On hobobiker the comment module is already disabled, so comments never render and there is nothing to close. Each prep step we keep is a script that prints a count.

---

## Three sites, side by side

| Project | Role |
|---|---|
| `hobobiker-copy` | the original, never touched |
| `hobobiker-prep` | rebuilt and prepared by scripts |
| `hobobiker-static` | the crawled HTML |

Note:
The prep changes are deliberate losses. Making them on the original would contaminate the reference. A prep copy rebuilt from a pristine restore every time can't drift into something nobody can reproduce.

---

## One command

restore → prepare → crawl → rewrite → verify

Note:
Same shape as last week's rule: scripts, not edits. On randyfay.com the whole pipeline runs in about 50 seconds. hobobiker is about ten times larger, so the crawl is pre-baked.

---

## A control run first

Crawl with **no** preparation

Nothing should fail

Note:
If a faithful crawl passes, any later failure was caused by a deliberate prep change. On randyfay.com: 490 of 490.

---

## HTTrack, wget, or a module?

* **HTTrack**: one-time export, any Drupal version
* **Static Generator**: inside Drupal 7, for ongoing changes
* **wget**: what we use

Note:
The article: "HTTrack is easier to set up for a one-and-done situation." A retired site is the one-and-done case. We use wget through the kit's mirror-static.mjs because it is already in the container, and the suite, not the crawler, decides whether it worked. Static Generator is Drupal 7 only and needs mod_rewrite.

---

## A crawler can't see everything

* Unlinked pages are invisible
* Prev/next links can explode: 9,738 pages

Note:
HTTrack "won't know anything about unlinked pages." The baseline was built by discovery too, so the archive and the baseline agree on what exists. Tripinfo per-day pages are bounded at one hop, the same decision as last week.

---

<!-- .slide: data-background-color="#02a8e2" -->

# The crawl is not self-contained

---

## Every content check passes

and the archive still depends on the old site

Note:
A crawler rewrites links only to pages it downloaded. Everything else stays an absolute URL on the crawled site: login links, feeds, comment permalinks. On randyfay.com, 4,084 references on 242 of 244 pages. None of them are in the baseline, so no content check sees them.

---

## The `static` tier

A page fails if it refers to:

* the site that was crawled
* a dead internal URL
* an external URL the original didn't have
* its own production domain

Note:
Reproduce, don't repair: a link that was already dead on the original may stay dead. A new CDN script means something dynamic crept in.

---

## The rewrite step

`rewrite-static.mjs`

* keep what the original had dead, made relative
* rules: anchor, unwrap, relative, remove
* everything else in the copy: relative

Note:
Deterministic and it prints a count per decision. hobobiker rehearsal: 291 of 2,454 pages changed. 232 dead or restricted references kept and made relative, 153 made relative, 222 links to pages not in the copy unwrapped.

---

<!-- .slide: data-background-color="#02a8e2" -->

# Decisions

---

## Rehearsal: 4,227 pass, 107 fail

Every semantic page passes

Note:
A quick wget mirror, not today's result. The 107 failures are decisions we have to make, not content we lost. None are allowlisted yet.

---

## Private pages: 403 or 404?

81 unpublished paths

Note:
A static archive has no unpublished pages, so they answer 404 where the original answered 403. Both are private. Declared in expected-differences.json scoped to "target": "static", so the Drupal 11 run next week still requires 403. A page that leaks still fails: "expected 403, got 200".

---

## Legacy files: 26 changed bytes

Old `.htm` pages under `files/`

Note:
wget's --convert-links rewrites links inside the old hand-made route maps and galleries, so their bytes no longer match the original. Either leave those files as they were, or declare it. Fix the right thing and say which it was.

---

## Every entry has a reason

`expected-differences.json`

Note:
Review every entry at the end. As narrow as the reason: whole-line matches, a single kind, a single target.

---

<!-- .slide: data-background-color="#02a8e2" -->

# Live

---

## Today

1. Plan the pipeline
2. Run it; read the failures
3. Make the archive self-contained
4. Declare what we changed on purpose
5. Green run, from a clean restore

Note:
Pre-baked: the full crawl and the browser tiers. Tag at the end: s2-static-green.

---

## Next week

**The Long Haul**: Drupal 11

Same tests

Note:
Oct 30. The static-only decisions don't apply: private pages must answer 403 again, and the stored tier checks the comments that never rendered.

---

## Resources / Sources

* Karen Stevenson, Lullabot: [Sending a Drupal Site Into Retirement](https://www.lullabot.com/articles/sending-a-drupal-site-into-retirement) · [Using HTTrack](https://www.lullabot.com/articles/sending-drupal-site-retirement-using-httrack) · [Using the Static Generation Module](https://www.lullabot.com/articles/sending-drupal-site-retirement-using-static-generation-module)
* Site source: [github.com/rfay/hobobiker.com](https://github.com/rfay/hobobiker.com)
* Files and sanitized database: [rfay/hobobiker-assets v1.1](https://github.com/rfay/hobobiker-assets/releases/tag/v1.1)
* The kit: [github.com/rfay/site-migration-kit](https://github.com/rfay/site-migration-kit)
* [ddev.com](https://ddev.com)

Note:
retirement-approach.md in the kit records which lessons we took from the series and how they show up in the tests.
