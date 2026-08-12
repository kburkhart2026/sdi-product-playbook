# SDI Product Development Playbook — GitHub-hosted setup

This folder is the GitHub-ready copy of the Playbook. It's a single HTML
file (`index.html`) with no build step and no server — GitHub Pages can
serve it as-is.

**What's in this copy:** all the same features as your working app, plus
the Margin Calculator (course library, per-program scenarios, "insert into
section" cards) and a cleaned-up toolbar (Tools ▾ / trimmed Export ▾ / Data
& Backup panel), already built and tested. This README covers the one part
that's yours to do: getting it actually hosted.

## The one thing to understand up front

**GitHub Pages (the part that serves the app itself) is always public** —
there's no per-viewer login gate, even on paid plans. That's fine, because
the app *shell* has no real data in it. What matters is that your
**content** stays restricted, and that's handled by a separate mechanism:

1. **App repo** (public) — this folder, hosted via GitHub Pages. Anyone can
   load the page; there's nothing sensitive in it.
2. **Content repo** (private) — one JSON file holding your actual content.
   Nobody can read or write it without a token.

Two kinds of access to that content repo, both **fine-grained GitHub
personal access tokens (PATs)** scoped to *only* that one repo — the
GitHub-native equivalent of an Azure SAS token:
- **Editor token** (you): Contents **Read and write**, pasted into this
  browser once via Export ▾ → Connect to GitHub (now under the ⚙ Data &
  Backup panel). Never shared.
- **Viewer token** (anyone you share a link with): Contents **Read-only**,
  carried in the link's `#data=<token>` fragment — never sent to a server or
  logged the way a `?query` parameter would be.

A structural bonus over a plain file share: every save becomes a real git
commit to the content repo. Version history, diffs, and rollback are native
to GitHub — no separate "push a timestamped backup" step needed, though the
existing daily local backup keeps working exactly as it does today,
independent of any of this.

## Part A — one-time GitHub setup (you do this)

1. Create a free GitHub account if you don't already have one.
2. Create **two repositories**:
   - `sdi-product-playbook` (public) — the app. This folder becomes that
     repo. (Any name works here — the app never references its own repo's
     name anywhere in its code, only the content repo's, below.)
   - `sdi-product-space-content` (private) — your data. Leave it empty; the
     app creates the content file itself the first time you use "Push local
     content to GitHub" (see One-time migration, below).
3. On the app repo: **Settings → Pages** → Source: the `main` branch, root
   folder. This gives you a URL like
   `https://<your-username>.github.io/sdi-product-playbook/`.
4. Push this folder to that repo. It's already a local git repository with
   one commit (`git log` shows `Initial GitHub-hosted app`) — you just need
   to point it at GitHub and push:
   ```bash
   cd "App - GitHub Hosted"
   git add -A
   git commit -m "Margin Calculator + toolbar cleanup"
   git remote add origin https://github.com/<your-username>/sdi-product-playbook.git
   git branch -M main
   git push -u origin main
   ```
   (Or skip the command line entirely — drag-and-drop `index.html` and
   `SDI-logo.png` into the repo on github.com and commit there instead.)
5. Generate **two fine-grained PATs**, both scoped to *only*
   `sdi-product-space-content` (Settings → Developer settings → Personal
   access tokens → Fine-grained tokens → Generate new):
   - **Editor token** (you): Repository permissions → Contents:
     **Read and write**. Longest expiration available. Keep this private —
     paste it into your own browser once, never into a shared link.
   - **Viewer token** (for others): Contents: **Read-only**. Whatever
     expiration you want. Generate a separate one per person/purpose if you
     want independent revocation later — deleting one token
     (Settings → Developer settings → PATs → the token → Delete) instantly
     cuts off just that link, without touching anyone else's.
6. Open `index.html` in a text editor and fill in the two blank constants
   near the top of the `<script>` block (search for `GITHUB_OWNER`):
   ```js
   const GITHUB_OWNER = '';           // <-- your GitHub username or org
   const GITHUB_CONTENT_REPO = '';    // <-- e.g. 'sdi-product-space-content'
   ```
   Commit and push that change.

## One-time content migration

Your real content currently lives in your browser's local storage, opened
from a different file path. Once Part A is done:

1. Open the new GitHub Pages URL (or this file locally, to test first).
2. **Export ▾ → Restore from backup** and load your most recent backup —
   the page opens with only starter content until you do this; it doesn't
   inherit anything from the copy you use day-to-day.
3. Open the **⚙ Data & Backup** panel → **Connect to GitHub…**, paste your
   editor PAT.
4. **⚙ Data & Backup** → **Push local content to GitHub** once, to seed the
   content repo for the first time.

From then on, every edit autosaves straight to GitHub — no more manual
pushes needed for content (only for app-code updates, see below).

## Sharing a read-only link

Take a viewer PAT from step 5 above and append it to your Pages URL as a
`#data=` fragment:
```
https://<your-username>.github.io/sdi-product-playbook/#data=<viewer-token>
```
Anyone opening that link sees current content, locked into view-only mode,
with nothing editor-only visible — just the program and **Export ▾** with
all seven export formats (PDF/Word/Excel per page, PDF/Word/PowerPoint/HTML
for the whole program).

## Shipping future app updates without touching content

The app repo and the content repo are different things. Pushing a new
commit here (an updated `index.html`) never touches the content repo, and
vice versa — editing content never triggers an app deploy.

## Before your first push

Two things in this folder shouldn't go to GitHub: `.DS_Store` (a macOS
artifact) and the `_pre_*_snapshot_*/` folders (local safety copies made
before edits, not meant to be part of the public app repo). A `.gitignore`
covering both is already included here — nothing further to do.

## Never put tokens in this file

**Don't paste PATs into this README, or into any file in this folder.**
This becomes the front page of your *public* app repo — anything committed
here is visible to anyone, forever (even after a later edit removes it, it
stays in git history). Neither token needs to live in a file at all:
- The **editor** token gets pasted once into the app itself (⚙ Data &
  Backup → Connect to GitHub…) and is then stored only in that browser's
  local storage.
- The **viewer** token only ever needs to exist inside a share link's
  `#data=` fragment (see "Sharing a read-only link," above) — copy it
  straight from GitHub's token page into the link, nowhere else.

If a token is ever typed into a file, treat it as compromised — delete it
on GitHub (Settings → Developer settings → PATs → the token → Delete) and
generate a fresh one, since deleting the file later doesn't undo the
exposure once it's been committed and pushed.

## What I can't test from here

I can't drive a real GitHub API call or an actual browser "Save as PDF"
dialog from this environment — both the GitHub connect/push flow and the
PDF export were verified as far as code review and mocked testing can go
(see prior session notes), but the first real end-to-end check — actually
connecting to GitHub, pushing, and opening a viewer link — is the one thing
only you can confirm once Part A is done.