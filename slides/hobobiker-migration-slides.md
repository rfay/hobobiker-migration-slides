# **Hobobiker Rides Again**

## Part 1: Road Test

Building the tests before the migration

![DDEV Logo](images/ddev-logo.svg)

Note:
Three live sessions. Today, no migration happens. We build the thing that tells us whether a migration worked.

---

## The series

* **Oct 16 · Road Test**: define success
* **Oct 23 · The Last Ride**: static HTML
* **Oct 30 · The Long Haul**: Drupal 11

Note:
Two very different migrations, one test suite. Everything today has to work for both.

---

## The site

hobobiker.com

* Drupal 6, last updated 15 years ago
* A bicycle trip from the Arctic to the tip of South America
* 1,296 nodes, 447 comments, 1,657 URL aliases

Note:
Frozen site. Nobody will edit it again. That makes some decisions easier later: one-time fixes are fine.

---

<!-- .slide: data-background-color="#02a8e2" -->

# The ideas

---

## Claude does its best work with

* A plan
* A clear view of source and destination
* Success criteria it can check itself

Note:
Most of the job is getting these three right. They are the same for any site. Today is mostly the third one.

---

## Define success first

Write the tests **before** touching anything

Freeze the result as a **baseline**

Note:
If the tests come after the migration, they quietly describe whatever the migration produced. Freeze the baseline, tag it, never regenerate it from inside the pipeline.

---

## Compare what a visitor sees

Not the markup

* Text, images, links, menus
* Status codes and paths

Note:
A static export and a Drupal 11 theme produce completely different HTML. A raw diff fails everything. So we record what the page says, and check that any target still says it.

---

## One suite, two targets

Static HTML · Drupal 11

Same tests, same baseline

Note:
If a test depends on how a target is built, it cannot judge both. That constraint keeps the tests honest.

---

## Reproduce, don't repair

* Broken on the old site → broken the same way
* Private on the old site → still private
* Log it in `DISCOVERIES.md`

Note:
"Not worth fixing" is not "not worth knowing." Fixing during a migration hides what the migration changed.

---

## Deliberate changes are declared

`expected-differences.json`

Every entry is a decision, with a reason

Note:
Example from the rehearsal: unpublished pages answer 404 in a static archive instead of 403. That is a decision, scoped to the static target only. A private page that leaks still fails.

---

## Scripts, not edits

restore → transform → crawl → verify

One command, from scratch, every time

Note:
Every script reports a count. A converter that changed 103 items yesterday and 0 today is telling you something broke upstream.

---

<!-- .slide: data-background-color="#02a8e2" -->

# Old sites surprise you

---

## What renders isn't what's stored

* PHP that runs at page view
* Macros: `[hobophoto:...]` (421 of them)
* Tables built by a custom module

Note:
Export the body column of a tripinfo page and you get one line of PHP. The table a visitor sees exists only at render time.

---

## What's stored doesn't render

* 447 comments: module disabled
* 85 photo pages: module gone
* 18 unpublished book pages

Note:
These return HTTP 200 or nothing at all, and look fine to a crawler. A static archive may leave them behind. A Drupal 11 rebuild should not.

---

## 9,738 pages

From a site with 1,296 nodes

Note:
Each per-day trip page has prev/next links that ignore which route it belongs to. Following them visits every day of every trip in about 18 contexts. The decision: capture the pages the route tables link to, don't follow prev/next. That decision is now one line of config.

---

## Check, don't assume

The September audit was wrong in about 10 places

Note:
MIGRATION_PREP.md was an earlier audit. On this copy the numbers had moved: published counts, which nodes use PHP, how many macros. The audit script re-checks every claim with a query.

---

<!-- .slide: data-background-color="#02a8e2" -->

# The setup

---

## A disposable copy

* Restores with one command (19 s)
* No production credentials, no route to production
* **Tell the agent**

Note:
An agent that doesn't know the blast radius will ask about every bulk UPDATE. Describe the environment accurately and it can work at the right pace. Only say it when it is true.

---

## Four layers of safety

1. Make it true
2. Tell the agent: `AGENTS.md`
3. Tell the classifier: `autoMode` in user settings
4. Tripwire hook

Note:
autoMode is only read from user settings, never from a repo, so a repo can't grant itself permissions. The tripwire hook is registered in settings.local.json and not committed: it blocks git push, which suits a disposable copy, not the real repo.

---

## The kit

`site-migration-kit`

* Vendored into the site repo
* One per-site file: `migration.config.mjs`

Note:
If reusing the kit on a new site needs a change anywhere else, that's a gap in the kit. The rehearsal found nine; they're fixed and pushed.

---

## The checks

* **semantic**: nothing missing
* **visible**: actually shown in a browser
* **layout**: same shape of page
* **access**, **assets**, **static**, **stored**

Note:
Semantic reads HTML and is fast. Visible runs a real browser and catches text that is present but hidden. Stored compares databases, for content that never rendered.

---

## Test the test

Break a copy on purpose

Every break must be caught

Note:
Delete a paragraph, an image, a file. Hide text with CSS. Make a private page public. Delete a comment from the database. A check never seen to fail is not trusted.

---

## Rehearsal results

* **10,613** checks pass on the original
* **6,732** pass on a static copy, 0 fail
* 7 of 7 deliberate breaks caught

Note:
The static copy was a quick wget mirror, not the real Part 2 migration. Its job is to prove the suite works against something other than the original.

---

<!-- .slide: data-background-color="#02a8e2" -->

# Live

---

## Today

1. Plan
2. Audit the source
3. Build the checks
4. Freeze the baseline
5. Break it on purpose

Note:
Slow steps are pre-baked: the Playwright image build, the browser-based checks, the full suite run.

---

## Next week

**The Last Ride**: static HTML

Same tests

Note:
Oct 23. Retire the site to static HTML following Karen Stevenson's Lullabot series, and run this suite against it.

---

## Links

* [github.com/rfay/site-migration-kit](https://github.com/rfay/site-migration-kit)
* [ddev.com](https://ddev.com)
