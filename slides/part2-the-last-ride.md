# **Hobobiker Rides Again**

## Part 2: The Last Ride

Retiring a site to static HTML

![DDEV Logo](images/ddev-logo.svg)

Note:
Today we turn hobobiker.com into a folder of plain HTML files, and use last week's test suite to prove nothing was lost.

---

## First: PHP2Shell

A PHP vulnerability this week lets one request run code on a server

You already know this, but fix your servers YESTERDAY.

These demo sites are public, so all three run fixed PHP

[ddev.com: what DDEV users need to know](https://ddev.com/blog/php2shell-what-ddev-users-need-to-know/)

Note:
CVE-2026-109618 and 109619, fixed in PHP 8.2.35 / 8.3.36 / 8.4.27 / 8.5.12 on Oct 21. Local DDEV projects aren't reachable from the internet, but these run on a Coder workspace with public URLs. The two Drupal 6 sites run PHP 7.3: the version number doesn't change, so check `ddev php -v` for "built: Oct 21 2026". The static site got the blog's Dockerfile.php2shell (8.4.27).

---

## Last week

* A frozen baseline: 2,345 pages
* **10,613** checks pass on the original
* 7 of 7 deliberate breaks caught

Note:
Tag s2-baseline-frozen. The suite doesn't know or care how today's target is built.

---

## Karen's Retirement Technique

*[Sending a Drupal Site Into Retirement](https://www.lullabot.com/articles/sending-a-drupal-site-into-retirement)*, Karen Stephenson, 2020

> "One of the biggest problems of transforming a dynamic site into static pages is that the URLs must change." <!-- .element: style="font-size: 0.6em" -->

Today there are alternatives, so this isn't necessarily a problem.

* Bare file host: `/a/b.html` or `/a/b/`
* GitHub, Cloudflare Pages: `/a/b`, but not `?page=2`
* nginx or Apache: every URL, even `?page=2`

Note:
Three articles: the approach, HTTrack, and the Static Generator module. Drupal 7 and GitHub Pages, but the thinking carries over. Our archive runs on nginx with one try_files line: /central_america_blog and /central_america_blog.html both answer, and so does /blog?page=1 (saved as the file "blog?page=1.html"). The archive's own links use the .html form because wget rewrote them; old bookmarks and search results use the bare form and still land. Plain static hosts ignore the query string, so /blog?page=1 would quietly show page 1; there the paginated URLs are the ones that really change, unless pagination is removed, as Karen suggests. The suite requests every URL exactly as the original served it, so it tells you which URLs a hosting choice breaks. Links on the last slide.

---

## Prepare before you crawl

Karen's checklist, checked against this hobobiker:

**To do:** turn off search · clear caches

**Already done or absent:** comments · contact form · login and author links · shortlinks · AJAX and filters

**Kept:** feeds, as frozen snapshots

Note:
Survey the site before applying the list. The crawl of the unprepared copy shows what's there: the search box is the only form in the whole archive (home page only, posting to index.html). Turning search off removes its block, and that absence is declared in expected-differences.json. Clearing caches: a cached rendering had the original's Coder hostname baked in, and the crawl copied it; the pristine restore clears caches anyway. Already done: the comment module is off (the 5 "leave a comment" links were already 404 on the original), the contact module is off (contact.html is a plain page), and there are no login, register or author links, no shortlink or generator tags, no AJAX or exposed filters. Feeds keep working as snapshots that never update. On randyfay.com, by contrast, "disable comments" would have deleted 670 real comments.

---

## The three projects

Live during the session:

* Original (Drupal 6): [hobobiker-copy](https://hobobiker-copy--remotework--rfay.coder.ddev.com)
* Prepared copy: [hobobiker-prep](https://hobobiker-prep--remotework--rfay.coder.ddev.com)
* Static archive: [hobobiker-static](https://hobobiker-static--remotework--rfay.coder.ddev.com)

Note:
Three sibling DDEV projects in one Coder workspace, each with a public URL during the session; they go away afterwards. The same links are in the github.com/rfay/hobobiker.com README.

---

## Three projects, five steps

1. **Restore** `hobobiker-prep` from the pristine dump (hobobiker-copy)
2. **Prepare** it: turn off search, flush cache
3. **Crawl** it into `hobobiker-static` with wget
4. **Rewrite** the crawl's links
5. **Check** `hobobiker-static` against the baseline from `hobobiker-copy`

Note:
The original, hobobiker-copy, is only the reference: the baseline came from it, and nothing changes it. Prep changes are deliberate losses, so they happen on hobobiker-prep, and restoring it from the pristine dump every time means it can't drift. hobobiker-static just serves the crawl. Rehearsal timings: restore 20 s, crawl about 2 minutes, rewrite 5 s, fast checks 52 s.

---

## Bug: Explicit links to hobobiker-copy or hobobiker.com

* Every content check passes, yet pages point back at the site we're retiring
* The `static` check finds those links; a rewrite step fixes them

Note:
A crawler rewrites links only to pages it downloaded. The static check fails a page that refers to the crawled site, a dead internal URL, or an external URL the original didn't have. rewrite-static.mjs makes the rest relative and prints a count per decision: 291 of 2,454 pages changed.

---

## Decisions

* **81** private pages: 404 instead of 403
* **26** old `.htm` files: wget rewrote their links
* **5** links to `/mexico/`: a folder with no index
* The search block: removed on purpose

Each one fixed, or declared with a reason

Note:
404 vs 403: both private; declared for the static target only, so Drupal 11 still needs 403, and a leaked page still fails. The .htm files: keep them byte for byte or declare it. /mexico/: old hand-made photo pages link with a trailing slash. Search: the prep step turns it off, so the home page loses its search block; one entry, as narrow as its reason. expected-differences.json, every entry reviewed at the end.

---

## Today

1. Plan the pipeline
2. Run it; read the failures
3. Make the archive stand alone
4. Declare what we changed on purpose
5. Green run from a clean restore

Note:
Pre-baked: the crawl and the browser checks. Tag at the end: s2-static-green. Next week (Oct 30), The Long Haul: Drupal 11, same tests.

---

## Resources

* Live during the session: [original](https://hobobiker-copy--remotework--rfay.coder.ddev.com) · [prep](https://hobobiker-prep--remotework--rfay.coder.ddev.com) · [static](https://hobobiker-static--remotework--rfay.coder.ddev.com)
* Karen Stevenson: [The approach](https://www.lullabot.com/articles/sending-a-drupal-site-into-retirement) · [HTTrack](https://www.lullabot.com/articles/sending-drupal-site-retirement-using-httrack) · [Static Generator](https://www.lullabot.com/articles/sending-drupal-site-retirement-using-static-generation-module)
* [PHP2Shell and DDEV](https://ddev.com/blog/php2shell-what-ddev-users-need-to-know/)
* Site source: [github.com/rfay/hobobiker.com](https://github.com/rfay/hobobiker.com)
* The kit: [github.com/rfay/site-migration-kit](https://github.com/rfay/site-migration-kit)
* [ddev.com](https://ddev.com)

Note:
Live during the session only: the three sites, linked from the github.com/rfay/hobobiker.com README.
