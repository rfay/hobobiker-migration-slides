# AGENTS.md

Slides for **Hobobiker Rides Again**, a three-part live series migrating hobobiker.com (Drupal 6)
to static HTML and to Drupal 11 with Claude. reveal.js, Markdown decks, published to GitHub Pages.

## Communication style

- Use direct, concise language without unnecessary adjectives or adverbs
- Avoid flowery or marketing-style language ("tremendous", "dramatically", "revolutionary", etc.)
- Don't include flattery or excessive praise ("excellent!", "perfect!", "great job!")
- State facts and findings directly without embellishment
- Skip introductory phrases like "I'm excited to", "I'd be happy to", "Let me dive into"

## Layout

| Part | Deck | Page | Published at |
|---|---|---|---|
| 1. Road Test (Oct 16) | `slides/hobobiker-migration-slides.md` | `index.html` | https://rfay.github.io/hobobiker-migration-slides/ |
| 2. The Last Ride (Oct 23) | `slides/part2-the-last-ride.md` | `part2.html` | https://rfay.github.io/hobobiker-migration-slides/part2.html |
| 3. The Long Haul (Oct 30) | `slides/part3-the-long-haul.md` | `part3.html` | https://rfay.github.io/hobobiker-migration-slides/part3.html |

- Each part has its own page, a copy of `index.html` with a different `data-markdown` file and
  `<title>`. A new page must also be added to the `cp` lines in `.github/workflows/deploy.yml`, or
  it 404s on the published site.
- Slides are separated by a line containing only `---`; speaker notes start with `Note:`.
- Pushing to `main` deploys. Preview locally with `npm start`: http://localhost:8000 and
  http://localhost:8000/part2.html (and part3.html). In the Coder workspace the same server is at
  https://8000--main--remotework--rfay.coder.ddev.com (private unless the port is shared publicly).

## Writing slides

- **Few, terse slides.** A slide carries a heading and a handful of short lines; the detail and the
  numbers' context go in the `Note:` block. A part is about a dozen slides, not two dozen.
- **Numbers come from measured runs**, not estimates: the hobobiker repo's `DISCOVERIES.md` and the
  kit's `webinar/PLAN.md` and `docs/`. Say which run they came from in the note.
- Credit sources by name and link them on the Resources slide (Karen Stevenson's Lullabot series for
  Part 2).
- Related repos: the site, `~/workspace/hobo/hobobiker-copy` (github.com/rfay/hobobiker.com); the kit,
  `~/workspace/site-migration-kit`. The site's README links these decks; keep the links in step.

## Git

Commit only when asked, and only the files for the change: leave unrelated edits (such as
`package-lock.json`) out. Never push; the maintainer pushes.

## Important Instruction Reminders

Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.
