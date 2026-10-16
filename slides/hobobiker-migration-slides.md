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

## The checks (or "Tiers")

* **semantic**: nothing missing from text
* **visible**: actually shown in a browser (playwright)
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

## Inside the checks

One question each

The rehearsal's finished suite:

```bash
ddev exec -d \
  /var/www/html/.tarballs/rehearsal/test/playwright bash
```

Note:
While the live agent works on its own suite, switch to a second terminal with the rehearsal's (a git worktree of tag rehearsal-baseline, gitignored, with node_modules). For each check: open the file it compares against, run a small subset (each takes seconds), look at what a failure says. Every command below runs in that shell. Full runs are too slow to run live: show the saved logs instead, e.g. tail tmp/selfcheck2.log (10,613 passed, 4.4 m). Deliberate breaks, 16 s: node scripts/negative-controls.mjs, then tmp/run-controls.sh /tmp/hobo-static tmp/c.log (11 pass) and tmp/run-controls.sh /tmp/hobo-neg tmp/b.log (9 fail, each named; grep Error: tmp/b.log). (`ddev playwright` doesn't work on this project: the add-on assumes the container starts in the project root, and this project's working_dir is docroot.)

---

## What a test looks like

From the rehearsal:

* **semantic**: [the test](https://github.com/rfay/hobobiker.com/blob/rehearsal-baseline/test/playwright/tests/semantic.spec.ts) · [what it checks on one page](https://github.com/rfay/hobobiker.com/blob/rehearsal-baseline/test/playwright/baseline/semantic/pages/node__4196.json)
* **layout**: [the test](https://github.com/rfay/hobobiker.com/blob/rehearsal-baseline/test/playwright/tests/layout.spec.ts) · [what it checks](https://github.com/rfay/hobobiker.com/blob/rehearsal-baseline/test/playwright/baseline/layout/layout.json)

Note:
Each spec file is a few lines that hand off to the kit; Playwright then generates one test per baseline page (2,436). The per-page record is the real content: node 4196's file holds the rendered tripinfo table as lines of text, and the test checks every line is still there. Extra content on the target is fine, missing content fails and is named. Layout: content column, sidebar, colors and fonts per page, measured in Chromium, compared with wide tolerances. The kit code behind them, if anyone asks: semantic tests/semantic-suite.mjs and lib/compare.mjs, layout tests/layout-suite.mjs and lib/layout.mjs, in site-migration-kit.

---

## semantic

Is every line, image, link and menu item still there?

`baseline/semantic/pages/node__4196.json`

```bash
npx playwright test --grep "semantic:.*@tripinfo"
```

Note:
Node 4196 is a route page. Its stored body is one line of PHP; the baseline record holds the 23-row table it renders. The check is one-directional: extra content on the target is fine, missing content fails and is named.

---

## visible

Can a visitor actually see it?

```bash
npx playwright test --grep "visible:.*@smoke"
```

Note:
Real Chromium, innerText. The negative control hides one paragraph on node 3830 with CSS: semantic passes (the text is in the HTML), visible fails. Known limit: an email address written by document.write is in no baseline.

---

## layout

Is the page the same shape?

`baseline/layout/layout.json`

```bash
npx playwright test --grep "layout:.*@smoke"
```

Note:
Not a pixel diff. Content column position and width, sidebar, header color, fonts, link color, measured in a browser, with wide tolerances. Catches a lost sidebar or a different theme.

---

## access

Is private still private?

81 unpublished paths, all 403

```bash
npx playwright test --grep "access:"
```

Note:
tests/access.spec.ts and baseline/manifest.json ("unpublished"). For a static target 404 is accepted (a decision in expected-differences.json); a leaked page still fails with "expected 403, got 200".

---

## assets

Same files, same bytes?

1,770 SHA-256 hashes

```bash
npx playwright test --grep "asset:"
```

Note:
tests/assets.spec.ts. Hashes only; the 1.2 GB of files is not copied into the baseline. Catches a file that went missing even when the page still links to it.

---

## static

Does the archive stand alone?

```bash
MIGRATION_TARGET=static \
  node kit/scripts/serve-static.mjs \
  --dir /tmp/hobo-static \
  --run "npx playwright test --grep static:"
```

Note:
Only for a static target, so it's next week's star. Here it runs against the rehearsal's quick wget mirror in the container's /tmp (2,436 pass, 16 s; gone after a ddev restart). Fails a page that still refers to the old site, a dead internal URL, or an external URL the original didn't have.

---

## stored

Did the database keep what never rendered?

`baseline/stored.json`

```bash
# host, in .tarballs/rehearsal/test/playwright
node kit/scripts/stored-exporters/drupal6.mjs \
  --out tmp/now.json
# container
STORED_TARGET_EXPORT=tmp/now.json \
  MIGRATION_TARGET=drupal11 \
  npx playwright test --grep "stored:.*@comments"
```

Note:
Compares baseline/stored.json with an export of the target's database (here, the original again: 148 comment threads pass). Enforced for Drupal 11 only. The negative control deletes one comment: "comment 1163 on node 4123 by Javier Martin is missing".

---

## baseline

Did the baseline catch the hard parts?

`tests/baseline-sanity.spec.ts`

```bash
npx playwright test --grep "baseline:"
```

Note:
Tests the baseline itself, not a target: tripinfo tables rendered (not PHP), every hobophoto page has images, empty-module pages recorded, no comments rendered, stored snapshot complete. A broken baseline would make every target look good.

---

## Next week

**The Last Ride**: static HTML

Same tests

Note:
Oct 23. Retire the site to static HTML following Karen Stevenson's Lullabot series, and run this suite against it.

---

## Resources / Sources

* **Live today only**: [the working copy](https://hobobiker-copy--remotework--rfay.coder.ddev.com)
* Site source: [github.com/rfay/hobobiker.com](https://github.com/rfay/hobobiker.com)
* Files and sanitized database: [rfay/hobobiker-assets v1.1](https://github.com/rfay/hobobiker-assets/releases/tag/v1.1)
* The kit: [github.com/rfay/site-migration-kit](https://github.com/rfay/site-migration-kit)
* [Playwright](https://playwright.dev) and the [ddev-playwright add-on](https://github.com/Lullabot/ddev-playwright)
* [ddev.com](https://ddev.com)

Note:
The assets release imports with the standard ddev import-db and ddev import-files, so anyone can reproduce this on their own machine.
