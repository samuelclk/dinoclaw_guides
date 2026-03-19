# Practical Guide: Setting Up `dinoclaw_work` with GitHub + Google Drive

This guide explains the practical setup we landed on for `dinoclaw_work`.

The goal is simple:

- keep **everything version-controlled in GitHub**
- keep **selected work accessible/editable in Google Drive**
- make it easy to manually edit files in Drive if needed
- still keep GitHub as the long-term history and rollback system

This is the layman version — no unnecessary theory, just the working model.

---

## The final architecture

We split responsibilities like this:

### GitHub = version control
GitHub stores:
- editable files
- working files
- final outputs
- full history
- rollback points

### Google Drive = access + manual editing surface
Google Drive stores:
- working files you may want to edit manually
- final deliverables you may want to share or permission differently

So the mental model is:

- **GitHub keeps the full history**
- **Google Drive gives you human-friendly access control and manual editing**

---

## The final folder model

Inside the `dinoclaw_work` repo, the final structure is:

- `Lido/`
  - `shared/`
  - `final/`
- `Stakesaurus/`
  - `shared/`
  - `final/`
- `Personal/`
  - `shared/`
  - `final/`
- `src/`

### What each part means

## `src/`
Use for:
- raw source files
- internal authoring files
- things not meant to be mirrored to Drive by default

## `<Category>/shared/`
Use for:
- working documents
- drafts
- files you may want to manually edit in Google Drive
- in-progress spreadsheets or docs

These go to:
- GitHub
- Google Drive

## `<Category>/final/`
Use for:
- polished deliverables
- final PDFs
- final spreadsheets
- final exported docs

These also go to:
- GitHub
- Google Drive

---

## The categories we created

Top-level categories:
- `Lido`
- `Stakesaurus`
- `Personal`

These sit **above** `shared` and `final`.

That means the correct shape is:

- `Lido/shared`
- `Lido/final`
- `Stakesaurus/shared`
- `Stakesaurus/final`
- `Personal/shared`
- `Personal/final`

This matters because it keeps work grouped by domain/person first, and only then by status.

---

## Why this structure is good

It gives you three layers:

### 1. Raw internal layer
- `src/`
- mostly GitHub/local only

### 2. Editable shared layer
- `<Category>/shared/`
- available in Drive for manual edits

### 3. Final delivery layer
- `<Category>/final/`
- available in Drive for polished outputs

This keeps the system practical:
- internal stuff stays internal
- editable shared files stay easy to reach
- final outputs stay neatly separated

---

## The authority rule

Default rule:
- **local repo / GitHub is the source of truth**

Exception:
- if Stakesaurus manually edits a file in Google Drive and explicitly says that version is authoritative,
  then Dino should:
  1. pull/export/download the Drive version
  2. replace the local file
  3. commit the updated version back into GitHub

This avoids confusion.

Without this rule, you end up wondering:
- is the GitHub version current?
- is the Drive version current?

So the policy is:
- **GitHub is default truth**
- **Drive can become temporary truth only when explicitly declared**

---

## The three repos we ended up with

We separated responsibilities across repositories.

### 1. Main operational repo
This is the live OpenClaw operational repo.

Use it for:
- config
- memory
- prompts
- scripts the system depends on
- recovery/runbook material

### 2. Guides repo
This stores human-facing guides and explanations.

Use it for:
- setup guides
- workflow docs
- postmortems
- polished writeups
- explanatory notes

### 3. Work repo: `dinoclaw_work`
This stores actual work products.

Use it for:
- editable work files
- outputs
- shared docs
- final deliverables

This split keeps the backup/config repo from turning into a notebook or document dump.

---

## GitHub auth we used

For GitHub, we did **not** rely on `gh auth` for this new repo flow.

Instead, we used a **shared Personal Access Token in `.env`**.

Example pattern:

```bash
GITHUB_SHARED_PAT=YOUR_PAT_HERE
GITHUB_GUIDES_REPO=https://github.com/YOUR_USERNAME/YOUR_GUIDES_REPO.git
GITHUB_OUTPUT_REPO=https://github.com/YOUR_USERNAME/YOUR_OUTPUT_REPO.git
```

Why this was better:
- it avoided conflicting with the existing GitHub auth already on the machine
- it let one PAT work for multiple repos
- it made command-specific authenticated Git pushes easy

Important note:
- if the PAT is **fine-grained**, it must have access to **all repos you want to use it with**

---

## Google Drive auth we used

Google Drive did **not** use a GitHub-style PAT.

It required:
- a Google Cloud project
- OAuth client credentials JSON
- `gog` authentication

### What we had to do

1. create/download Google OAuth client credentials
2. register them with `gog`
3. authenticate the Google account
4. switch `gog` keyring backend to `file`

### The important fix

On this machine, `gog`'s default keyring behavior failed.

The working fix was:

```bash
gog auth keyring file
```

That made `gog` store tokens in its own encrypted file-based keyring.

---

## The passphrase lesson

When using `gog` file keyring, it may prompt for:

```bash
Enter passphrase to unlock "/home/huatyou/.config/gogcli/keyring"
```

For automation, the practical fix was to put this in `.env`:

```bash
GOG_KEYRING_PASSWORD=YOUR_PASSPHRASE_HERE
```

That lets automated `gog` commands run without stopping for a prompt.

Important tradeoff:
- this is convenient
- but the passphrase now exists in plaintext inside `.env`

That may still be acceptable if:
- `.env` is local only
- permissions are locked down
- it is never committed to Git

---

## The Google API enablement lesson

Even after auth succeeded, Drive actions initially failed with:

- `Google API error (403 accessNotConfigured)`

That happened because the APIs were not enabled yet in Google Cloud.

The fix was to enable:
- Google Drive API
- Google Docs API
- Google Sheets API

So the complete lesson is:
- OAuth credentials alone are **not enough**
- the actual APIs must also be enabled in the Google Cloud project

---

## The folder-ID lesson

When we first created Drive folders, shell parsing of JSON output was sloppy.

The folders were created, but the IDs were not being captured reliably.

That is dangerous if you want real automation.

So the fix was:
- create a durable mapping file in the work repo:

```bash
config/google-drive-folders.json
```

This file stores:
- the current Drive root folder ID
- each category folder ID
- each `shared` and `final` folder ID
- the Drive web links

That became the source of truth for Drive-side automation.

This is much better than trying to re-parse ad hoc shell output every time.

---

## The hierarchy mistake we corrected

We first created the wrong hierarchy:

- `shared/Lido`
- `shared/Stakesaurus`
- `shared/Personal`
- `final/Lido`
- `final/Stakesaurus`
- `final/Personal`

That was not what you wanted.

The correct hierarchy is category-first:

- `Lido/shared`
- `Lido/final`
- `Stakesaurus/shared`
- `Stakesaurus/final`
- `Personal/shared`
- `Personal/final`

So the practical lesson is:
- when defining folder structures, confirm the parent/child hierarchy explicitly before automating creation on both GitHub and Drive

This is especially important because fixing hierarchy later is annoying.

A related practical lesson:
- if a bad automation run creates folders at Drive root by mistake, clean them up right away so your root stays readable and you do not confuse them with the real system

---

## The Drive replacement lesson

After creating the corrected Drive structure, we replaced the old one by:
- renaming the new correct root to the canonical name
- trashing the old incorrect root
- removing leftover stray root-level folders accidentally created during the broken first pass

That is cleaner than trying to mentally track:
- old wrong tree
- new correct tree
- stray folders at Drive root

So the practical lesson is:
- once the corrected tree exists and the mapping file is updated, replace the old structure quickly and clean up any root-level leftovers immediately

---

## What the final working setup now looks like

### GitHub
GitHub stores the full work repo, including:
- category folders
- shared files
- final files
- source files
- scripts
- folder mapping config
- version history

### Google Drive
Google Drive stores the accessibility/editability layer:
- one clean root folder: `dinoclaw_work`
- category folders inside that root
- each category's `shared/`
- each category's `final/`

### Default behavior
- GitHub/local is default source of truth
- Drive is used for access control and manual edits

### Override behavior
- if Stakesaurus edits something manually in Drive and says it is authoritative,
  Dino syncs it back into GitHub

---

## The simplest way to think about it

If you do not want to overthink it:

### Use GitHub for
- everything you want version history for
- everything you do not want to lose
- everything you may want to roll back later

### Use Google Drive for
- files you want to open easily
- files you want to share with controlled permissions
- files you may want to manually edit in the browser or Drive apps

### Use category folders for
- who/what the work belongs to

### Use `shared` for
- editable in-progress work

### Use `final` for
- polished outputs

That is the practical model.

---

## Practical setup checklist

If you want to reproduce this from scratch:

1. create the `dinoclaw_work` GitHub repo
2. create a local work repo
3. authenticate GitHub with a PAT stored in `.env`
4. create the folder layout:
   - `Lido/shared`
   - `Lido/final`
   - `Stakesaurus/shared`
   - `Stakesaurus/final`
   - `Personal/shared`
   - `Personal/final`
   - `src`
5. push the repo to GitHub
6. create a Google Cloud OAuth client
7. authenticate `gog`
8. set `gog` keyring backend to `file`
9. put `GOG_KEYRING_PASSWORD` in `.env` if you want automation
10. enable Drive/Docs/Sheets APIs
11. create the matching Drive folder tree
12. save Drive folder IDs into `config/google-drive-folders.json`
13. treat GitHub as default truth, unless a Drive file is explicitly declared authoritative

---

## Final takeaway

The final system is good because it combines:

- **GitHub** for history, structure, rollback, and discipline
- **Google Drive** for accessibility, sharing, manual edits, and permission control

That is the balance we were trying to get.

The clean mental model is:

- **GitHub = memory + history**
- **Drive = access + human editing surface**
- **GitHub stays default truth unless you explicitly promote a Drive edit back into it**

That’s the working setup.
