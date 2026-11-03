# justducky.au

The family hub page. One prominent link to [Ducky's
Garden](https://github.com/greendragon5000/charlies-garden) at `/Plants/`,
with room to add other hobbies later (cooking, etc.). Plain HTML/CSS, no
build step, no framework - the same approach as the garden site.

That link said `/garden/` until [CHA-103](/CHA/issues/CHA-103) on 3 Nov
2026. `/garden/` still resolves - it is a permanent 301 to `/Plants/`, put
there by [CHA-74](/CHA/issues/CHA-74)'s cutover so that already-printed QR
tags keep working - so this was a redirect hop removed, not a break
repaired.

This repo is **just the hub page**: `index.html`, `style.css`, and the
`.htaccess`/`.cpanel.yml` that deploy them to the site's document root. It
does not contain the garden app, and must never contain a `Plants/`
directory - see "Why no `Plants/` directory here" below.

Parent plan: see the garden repo's README and
[CHA-59](/CHA/issues/CHA-59#document-plan) for why `justducky.au/` now
hosts a hub instead of redirecting outright to `garden.justducky.au`.

## public_html is shared - the deploy rules that matter

`~/public_html` (this repo's `DEPLOYPATH`) is **not exclusive to this
repo**. It also holds:

- `Plants` - a symlink to `~/garden.justducky.au/public`, which is how
  `justducky.au/Plants/` serves the garden app without copying any of its
  code into `public_html`
- `garden/` - an ordinary directory holding nothing but an `.htaccess`
  that 301s every path under it to the same path under `/Plants/`, query
  string preserved. That redirect **is** the already-printed QR tags
  continuing to work, so it is load-bearing infrastructure and must not be
  removed or weakened - see the garden repo's README, "Printing new tags".
- any future hobby folder (`cooking/`, etc.)

**These two swapped places on 3 Nov 2026**
([CHA-74](/CHA/issues/CHA-74)). Before the cutover, `Plants/` was the old
spreadsheet-driven site's QR-tag pages and `garden` was the symlink to the
app. Whatever was still in the old `Plants/` was moved rather than
deleted, to `~/legacy-data/Plants-dir-2026-11-03/`. Any older note that
calls `garden` a symlink is describing the pre-cutover layout.

Because of that:

- **`.cpanel.yml` copies explicit files by name, never `cp -Rf .`.** A
  recursive copy of the whole repo would also dump `README.md`,
  `.gitignore`, and this repo's own `.git` into the live, public
  `public_html` - and a stray `.git` there turns every later deploy into a
  permission error (see the garden repo's `.cpanel.yml` for the long
  version of that failure mode). Naming the files also means nothing here
  needs to delete a stray `.git` after copying, the way the garden repo's
  deploy does.
- **`.cpanel.yml` never runs `rm -rf` against `DEPLOYPATH` or anything
  under it.** `public_html` is where the `Plants` symlink and the
  `garden/` redirect directory live. Deleting that directory, or even
  clearing it first, takes them out with it - and taking out `Plants`
  takes the whole garden app off the web. See
  [CHA-17](/CHA/issues/CHA-17) for what that costs.
- **`.htaccess` has no catch-all rewrite.** It only forces HTTPS and the
  canonical host (`justducky.au`, not `www.` or plain `http://`) and
  leaves every request's path untouched. A catch-all rewrite to a front
  controller - the kind the garden app's own `public/.htaccess` uses -
  would swallow requests meant for `/garden/` or `/Plants/` before they
  ever got there, since this hub has no router to hand them back to.

### Why no `Plants/` directory here

If this repo ever had a real `Plants/` directory, `cp -f` (or any
recursive copy) would write *through* the `Plants` symlink in
`public_html` and land files inside the live garden app's `public/`
instead of creating a folder next to it. The hub only ever *links* to
`/Plants/`; it never *contains* one.

Before 3 Nov 2026 this warning named `garden/`, because that was the
symlink. **The hazard moved with the symlink**: a `garden/` directory in
this repo would now merely shadow the redirect, while a `Plants/` one
would write into the live app.

## The hobby convention

Every hobby area under `justducky.au/` is one of exactly two things:

1. **A plain directory in this repo** - static content with no server-side
   app behind it (a future `/cooking/` is the expected example). Add the
   directory, add its files to the `cp -f` line in `.cpanel.yml`, and link
   to it from `index.html`.
2. **A symlink in `public_html` pointing at an app's own `public/`
   directory, which lives outside `public_html` entirely.** This is how
   `/Plants/` works: `~/public_html/Plants` is a symlink to
   `~/garden.justducky.au/public`, and the app's `lib/`, `templates/`,
   `config.php` and `data/` all stay above the web root, in
   `~/garden.justducky.au/`, where they are not web-reachable.

An app's non-public code, config, or data **never** goes inside
`public_html` - not even temporarily. Writing this down now is what makes
adding `/cooking/` later a ten-minute job (a static directory) rather than
a rethink, and what stops a future app from accidentally exposing its
`config.php` the way option 1 above cannot.

**`/cooking/` is not a live link, despite what `grep` suggests.**
`index.html` carries it only as a commented-out example inside the "add
future hobbies here" block, so nothing a visitor can click points at it,
and `https://justducky.au/cooking/` returning a 404 is not a reachable
dead link. The catch is that `curl https://justducky.au/ | grep -oE
'href="[^"]*"'` happily matches inside HTML comments and reports
`href="/cooking/"` anyway - which is what made
[CHA-103](/CHA/issues/CHA-103) go looking for a broken front-door link
that was never there. Check whether a match is commented out before acting
on it.

The old `FoodGardenPlantingCalendar/` page (unfinished, never wired up to
a data source) is being retired along with the rest of the old site - see
[CHA-62](/CHA/issues/CHA-62) for the archive. If a planting calendar comes
back later, it comes back as a proper `/calendar/` hobby area under this
convention, not as a restored copy of that stub.

## Deploying (the owner runs this in cPanel)

**This repo is live, and it is the only place to change the hub page.**
The clone cPanel deploys from is `~/repositories/justducky-au`, confirmed
by the owner on 3 Nov 2026. The first deploy landed on or before 31 Oct
2026 - the deployed `index.html`, `style.css` and `.htaccess` in
`~/public_html` all carry a 31 Oct 19:22 timestamp, which is the signature
of a deploy rather than hand-editing.

That matters for anyone tempted to fix something quickly on the server:
editing `~/public_html/index.html` in place **works until the next deploy,
then silently reverts**, and the reversion looks like a mystery because
nothing records that a deploy overwrote it. Changes belong here.

To release a change:

1. Push to `main` on GitHub.
2. **Known cPanel quirk** (already hit once on the garden repo): after
   pushing to GitHub, the Git Version Control page needs a manual
   **refresh** before the *Deploy HEAD Commit* button appears under
   Manage - it doesn't show up automatically right after a push.
3. *Update from Remote*, then *Deploy HEAD Commit*. `.cpanel.yml` copies
   `index.html`, `style.css` and `.htaccess` into `~/public_html` -
   nothing else, and nothing is deleted first.

   **The two buttons have two different timings.** *Update from Remote*
   moves the clone's HEAD and is instant, so `git -C
   ~/repositories/justducky-au log --oneline -1` shows the new commit
   straight away. *Deploy HEAD Commit* only **queues** the copy, which
   runs on cPanel's own schedule. In the gap, the clone is at the new
   commit while the live page still serves the old file. So verify by
   re-checking the URL until it flips, not by one immediate reading -
   only a stale result that *persists* for a few minutes means the deploy
   failed, and the symptom then is a deployment stuck in `queued` rather
   than any visible error.
4. **Check `DirectoryIndex` ordering** if an old home page ever starts
   winning over `index.html` - cPanel/Apache's default index order
   sometimes tries `index.php` or `default.html` first.
5. Verify: `https://justducky.au/` shows the hub, its Ducky's Garden link
   reads `href="/Plants/"` and lands on a `200`, and
   `https://justducky.au/garden/...` still 301s to the matching `/Plants/`
   path - this deploy must not have touched either.

This repo intentionally does not set up the `Plants` symlink, the
`garden/` 301, or the subdomain redirect - that's cPanel account work
covered by separate tasks ([CHA-64](/CHA/issues/CHA-64),
[CHA-74](/CHA/issues/CHA-74)), not something `.cpanel.yml` can or should
do.
