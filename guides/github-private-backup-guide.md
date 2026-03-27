# Safe Backup Guide: `.openclaw` → Private GitHub Repo

This guide documents the safest practical setup for backing up the important parts of `~/.openclaw` to a **private GitHub repository** while keeping live secrets out of Git.

---

## Goal

Back up `~/.openclaw` to a **private GitHub repo** with:

- version history
- rollback to earlier commits
- a daily automated backup
- secrets kept outside Git via `~/.openclaw/.env`

---

## What this setup protects against

This setup is meant to preserve:

- `workspace/`
- `openclaw.json`
- `cron/`
- selected safe parts of `.openclaw`

This setup is **not** meant to store live credentials in GitHub.

---

## 1. Create a GitHub account

If you do not already have one:

1. Go to <https://github.com/signup>
2. Create an account
3. Verify your email
4. Enable **2FA** in GitHub settings

If you already have a GitHub account, use that.

---

## 2. Create a new private GitHub repo

1. Go to <https://github.com/new>
2. Choose a repository name, e.g. `dinoclaw`
3. Set visibility to **Private**
4. You can initialize with a README or leave it empty

If you initialize with a README, that is fine — you can replace the repo history later if needed.

Example repo URL:

```bash
https://github.com/YOUR_USERNAME/dinoclaw.git
```

---

## 3. Create a GitHub Personal Access Token (PAT)

Use a **fine-grained PAT** if you want to restrict access to a single repo.

### Create the token

1. Go to <https://github.com/settings/personal-access-tokens>
2. Create a **Fine-grained token**
3. Set **Repository access** → **Only select repositories**
4. Choose your private backup repo only

### Recommended permissions

Repository permissions:

- **Contents: Read and write**
- **Pull requests: Read and write** *(optional, useful if you will use PR workflows)*
- **Workflows: Read and write** *(only if you expect to modify `.github/workflows/*`)*

### Important

- Do **not** paste the PAT into chat
- Paste it only into terminal prompts on the machine that needs it
- Treat the PAT like a password

---

## 4. Authenticate GitHub CLI (`gh`)

Install GitHub CLI if needed, then authenticate.

First change to your home directory:

```bash
cd ~
```

Then run:

```bash
gh auth login
```

Choose:

- **GitHub.com**
- **HTTPS**
- **Authenticate Git with your GitHub credentials?** → **Yes**
- **How would you like to authenticate GitHub CLI?** → **Paste an authentication token**

Then paste your PAT **into the terminal prompt**.

Verify:

```bash
gh auth status
```

---

## 5. Configure Git identity

Set your Git name and email so commits work:

```bash
git config --global user.name "YOUR_GITHUB_USERNAME"
git config --global user.email "YOUR_EMAIL@example.com"
```

If you prefer privacy, you can use your GitHub noreply email.

---

## 6. Create `~/.openclaw/.env` for secrets

Do **not** keep live secrets directly in `openclaw.json` if you plan to back it up.

Change into the OpenClaw directory:

```bash
cd /home/<user>/.openclaw
```

Open the file in `nano`:

```bash
nano .env
```

Paste contents like:

```bash
OLLAMA_API_KEY=YOUR_OLLAMA_API_KEY
TELEGRAM_BOT_TOKEN=YOUR_TELEGRAM_BOT_TOKEN
OPENCLAW_GATEWAY_TOKEN=YOUR_OPENCLAW_GATEWAY_TOKEN
```

Save and exit `nano` using:

- `CTRL+O` to write the file
- `ENTER` to confirm the filename
- `CTRL+X` to exit

Then set safe permissions:

```bash
chmod 600 ~/.openclaw/.env
```

### Important

- `.env` must **not** be committed to Git
- `.env` does **not** auto-load by magic
- your OpenClaw service must be configured to load it

---

## 7. Make the OpenClaw systemd service load `.env`

If OpenClaw runs as a **user systemd service**, add a drop-in override.

From anywhere in the terminal, run:

```bash
systemctl --user edit openclaw-gateway
```

An editor will open. Put this in the editor:

```ini
[Service]
EnvironmentFile=%h/.openclaw/.env
```

If the editor is `nano`, save and exit with:

- `CTRL+O`
- `ENTER`
- `CTRL+X`

Then reload and restart:

```bash
systemctl --user daemon-reload
systemctl --user restart openclaw-gateway
systemctl --user status openclaw-gateway --no-pager
```

---

## 8. Convert secret values in `openclaw.json` to env-backed refs

Instead of plaintext secrets like this:

```json
"botToken": "123:abc"
```

use env-backed SecretRefs like this:

```json
"botToken": {
  "source": "env",
  "provider": "default",
  "id": "TELEGRAM_BOT_TOKEN"
}
```

### Example secret fields to convert

- `models.providers.ollama.apiKey` → `OLLAMA_API_KEY`
- `channels.telegram.botToken` → `TELEGRAM_BOT_TOKEN`
- `gateway.auth.token` → `OPENCLAW_GATEWAY_TOKEN`

### Example structure

```json
"apiKey": {
  "source": "env",
  "provider": "default",
  "id": "OLLAMA_API_KEY"
}
```

```json
"botToken": {
  "source": "env",
  "provider": "default",
  "id": "TELEGRAM_BOT_TOKEN"
}
```

```json
"token": {
  "source": "env",
  "provider": "default",
  "id": "OPENCLAW_GATEWAY_TOKEN"
}
```

After converting, restart OpenClaw if needed and verify it still starts cleanly.

---

## 9. Initialize a Git repo at `~/.openclaw`

Change into the top-level OpenClaw folder first:

```bash
cd /home/<user>/.openclaw
git init
git branch -M main
```

If you previously had a repo only inside `workspace/`, treat that as a separate nested repo and migrate carefully.

---

## 10. Add a safe `.gitignore`

Change into the repo root:

```bash
cd /home/<user>/.openclaw
```

Then open `.gitignore` in `nano`:

```bash
nano .gitignore
```

Recommended baseline:

```gitignore
# Secrets / credentials
.env
.env.*
credentials/
devices/
telegram/

# Runtime / generated state
logs/
completions/
delivery-queue/
canvas/
media/
update-check.json

# OS/editor noise
.DS_Store
Thumbs.db
*~
*.swp
*.swo

# Temp/cache
*.log
*.tmp
*.temp
__pycache__/
node_modules/

# Nested repo metadata
workspace/.git/

# OpenClaw sensitive/runtime state
.git.backup-workspace-*/
agents/*/sessions/
agents/*/agent/auth-profiles.json
identity/
memory/*.sqlite
openclaw.json.bak*
```

Save and exit `nano` with:

- `CTRL+O`
- `ENTER`
- `CTRL+X`

### Why these are ignored

Because they may contain:

- secrets
- device/session state
- credentials
- runtime logs
- caches
- transcripts
- local rollback clutter

---

## 11. Add the repo remote and push

Point the repo at your private GitHub repo:

```bash
cd /home/<user>/.openclaw
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
```

If `origin` already exists:

```bash
git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
```

First push:

```bash
git add .
git commit -m "Initialize top-level .openclaw backup repo"
git push -u origin main
```

If you intentionally want to replace the remote repo entirely with this new local history:

```bash
git push -u origin main --force
```

Use force push only when you mean it.

---

## 12. Everyday manual backup commands

Use this sequence:

```bash
cd /home/<user>/.openclaw
git add .
git diff --cached --quiet && echo "No changes to commit" || git commit -m "Backup update"
git push
```

This avoids an error when there is nothing new to commit.

---

## 13. Automate daily backup with a bash script

Create the scripts folder and change into it:

```bash
mkdir -p /home/<user>/.openclaw/scripts
cd /home/<user>/.openclaw/scripts
```

Then create the script in `nano`:

```bash
nano backup-openclaw.sh
```

Paste these contents:

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO_DIR="/home/<user>/.openclaw"
LOG_DIR="$REPO_DIR/logs"
LOG_FILE="$LOG_DIR/backup-openclaw.log"

mkdir -p "$LOG_DIR"

{
  echo "[$(date --iso-8601=seconds)] Starting daily backup"
  cd "$REPO_DIR"

  git add .

  if git diff --cached --quiet; then
    echo "[$(date --iso-8601=seconds)] No changes to commit"
  else
    git commit -m "Daily backup update"
    git push
    echo "[$(date --iso-8601=seconds)] Backup pushed successfully"
  fi
} >> "$LOG_FILE" 2>&1
```

Save and exit `nano` with:

- `CTRL+O`
- `ENTER`
- `CTRL+X`

Then make it executable:

```bash
chmod 700 /home/<user>/.openclaw/scripts/backup-openclaw.sh
```

---

## 14. Add a daily cron job

Edit your user crontab:

```bash
crontab -e
```

If prompted to choose an editor, choose `nano` for simplicity.

Add this line:

```cron
15 4 * * * /home/<user>/.openclaw/scripts/backup-openclaw.sh
```

This runs the backup daily at **4:15 AM**.

If you are using `nano`, save and exit with:

- `CTRL+O`
- `ENTER`
- `CTRL+X`

Verify:

```bash
crontab -l
```

---

## 15. Test the backup script manually

Before trusting automation, change into the repo root and run it once yourself:

```bash
cd /home/<user>/.openclaw
./scripts/backup-openclaw.sh
```

Then inspect:

```bash
git status
cat /home/<user>/.openclaw/logs/backup-openclaw.log
```

---

## 16. Ongoing maintenance

### Periodically check:

```bash
cd /home/<user>/.openclaw
git status
git remote -v
git log --oneline -n 5
```

### If `.env` changes

No Git action is needed, because `.env` is ignored.

### If `openclaw.json` changes

It will be backed up as long as secrets remain env-backed refs and not plaintext.

---

## 17. What should stay out of Git

Keep these out of the repo unless you have a very specific reason:

- `.env`
- `credentials/`
- `devices/`
- `telegram/`
- `identity/`
- `agents/*/sessions/`
- `agents/*/agent/auth-profiles.json`
- `memory/*.sqlite`
- `openclaw.json.bak*`
- runtime logs / queues / caches

---

## 18. Summary

The safe pattern is:

1. Put secrets in `~/.openclaw/.env`
2. Make systemd load `.env`
3. Use env-backed SecretRefs in `openclaw.json`
4. Ignore secret/runtime folders in `.gitignore`
5. Back up `~/.openclaw` to a **private GitHub repo**
6. Automate commits and pushes with a daily cron job

That gives you a simple, versioned, rollback-friendly offsite backup without dumping live secrets into GitHub.
