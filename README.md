# justducky.au

The family hub page. One prominent link to [Ducky's
Garden](https://github.com/greendragon5000/charlies-garden) at `/garden/`,
with room to add other hobbies later (cooking, etc.). Plain HTML/CSS, no
build step, no framework - the same approach as the garden site.

This repo is **just the hub page**: `index.html`, `style.css`, and the
`.htaccess`/`.cpanel.yml` that deploy them to the site's document root. It
does not contain the garden app, and must never contain a `garden/`
directory - see "Why no `garden/` directory here" below.

Parent plan: see the garden repo's README and
[CHA-59](/CHA/issues/CHA-59#document-plan) for why `justducky.au/` now
hosts a hub instead of redirecting outright to `garden.justducky.au`.

## public_html is shared - the deploy rules that matter

`~/public_html` (this repo's `DEPLOYPATH`) is **not exclusive to this
repo**. It also holds:

- `Plants/` - the old spreadsheet-driven site's QR-tag pages, left in
  place because they're printed on tags in the garden (see the garden
  repo's README, "Redirecting the old site to the new one")
- `garden` - a symlink to `~/garden.justducky.au/public`, which is how
  `justducky.au/garden/` serves the garden app without copying any of its
  code into `public_html`
- any future hobby folder (`cooking/`, etc.)

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
  under it.** `public_html` is where `Plants/` and the `garden` symlink
  live. Deleting that directory, or even clearing it first, takes them out
  with it. See [CHA-17](/CHA/issues/CHA-17) for what that costs.
- **`.htaccess` has no catch-all rewrite.** It only forces HTTPS and the
  canonical host (`justducky.au`, not `www.` or plain `http://`) and
  leaves every request's path untouched. A catch-all rewrite to a front
  controller - the kind the garden app's own `public/.htaccess` uses -
  would swallow requests meant for `/garden/` or `/Plants/` before they
  ever got there, since this hub has no router to hand them back to.

### Why no `garden/` directory here

If this repo ever had a real `garden/` directory, `cp -f` (or any
recursive copy) would write *through* the `garden` symlink in
`public_html` and land files inside the live garden app's `public/`
instead of creating a folder next to it. The hub only ever *links* to
`/garden/`; it never *contains* one.

## The hobby convention

Every hobby area under `justducky.au/` is one of exactly two things:

1. **A plain directory in this repo** - static content with no server-side
   app behind it (a future `/cooking/` is the expected example). Add the
   directory, add its files to the `cp -f` line in `.cpanel.yml`, and link
   to it from `index.html`.
2. **A symlink in `public_html` pointing at an app's own `public/`
   directory, which lives outside `public_html` entirely.** This is how
   `/garden/` works: `~/public_html/garden` is a symlink to
   `~/garden.justducky.au/public`, and the app's `lib/`, `templates/`,
   `config.php` and `data/` all stay above the web root, in
   `~/garden.justducky.au/`, where they are not web-reachable.

An app's non-public code, config, or data **never** goes inside
`public_html` - not even temporarily. Writing this down now is what makes
adding `/cooking/` later a ten-minute job (a static directory) rather than
a rethink, and what stops a future app from accidentally exposing its
`config.php` the way option 1 above cannot.

## Deploying (not done yet - the owner runs this)

This task is code-only. No deploy has happened. The cPanel work that
follows, in roughly this order:

1. cPanel -> **Git Version Control** -> create the clone for this repo
   **outside** `public_html` (cPanel clones to a path you choose; pick
   something like `~/justducky-au-repo`, not `~/public_html` itself).
2. **Known cPanel quirk** (already hit once on the garden repo): after
   pushing to GitHub, the Git Version Control page needs a manual
   **refresh** before the *Deploy HEAD Commit* button appears under
   Manage - it doesn't show up automatically right after a push.
3. Deploy HEAD Commit. `.cpanel.yml` copies `index.html`, `style.css` and
   `.htaccess` into `~/public_html` - nothing else, and nothing is deleted
   first.
4. **Check `DirectoryIndex` ordering** if the old home page keeps winning
   over `index.html` - cPanel/Apache's default index order sometimes tries
   `index.php` or `default.html` first. If the old site left either of
   those behind in `public_html`, either delete it or confirm
   `DirectoryIndex` lists `index.html` ahead of it.
5. Verify: `https://justducky.au/` shows the hub, the Ducky's Garden link
   works, and `https://justducky.au/Plants/...` (the old QR-tag pages)
   still serves - this deploy must not have touched it.

This repo intentionally does not set up the `garden` symlink, the
subdomain redirect, or retire the old site - that's cPanel account work
covered by a separate task ([CHA-64](/CHA/issues/CHA-64)), not something
`.cpanel.yml` can or should do.
