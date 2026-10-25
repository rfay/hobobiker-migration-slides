# Hobobiker Rides Again: slides

Slides for **Hobobiker Rides Again**, a three-part live series in which hobobiker.com (Drupal 6) is
migrated to static HTML and to Drupal 11 with Claude, using
[site-migration-kit](https://github.com/rfay/site-migration-kit).

| Part | Slides | Deck source |
|---|---|---|
| 1. Road Test (Oct 16) | **[View](https://rfay.github.io/hobobiker-migration-slides/)** | `slides/hobobiker-migration-slides.md`, page `index.html` |
| 2. The Last Ride (Oct 23) | **[View](https://rfay.github.io/hobobiker-migration-slides/part2.html)** | `slides/part2-the-last-ride.md`, page `part2.html` |
| 3. The Long Haul (Oct 30) | **[View](https://rfay.github.io/hobobiker-migration-slides/part3.html)** | `slides/part3-the-long-haul.md`, page `part3.html` |

The decks are [reveal.js](https://revealjs.com/) Markdown; press `S` for speaker notes. Preview locally
with `npm start` and open http://localhost:8000 (Part 1), http://localhost:8000/part2.html (Part 2) or
http://localhost:8000/part3.html (Part 3).
Pushing to `main` deploys to GitHub Pages. A new part's page must also be copied into the site in
`.github/workflows/deploy.yml`.
