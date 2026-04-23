# gstack Setup Guide for Cursor — Windows / macOS / Linux

**Goal:** Install gstack globally so its 30+ planning / review / QA skills are available in every Cursor project on your machine.

**Why gstack:** Turns Cursor into a structured AI team (CEO / Eng Manager / Designer / QA / Release Manager) that runs a proper sprint: *Think → Plan → Build → Review → Test → Ship*.

---

## 1. Known issue: README is ahead of the code (all platforms)

The gstack README lists `--host cursor` as a supported option. The actual `setup` script in `main` does **not** support it yet — only: `claude`, `codex`, `kiro`, `factory`, `opencode`, `openclaw`, `hermes`, `gbrain`, `auto`.

**Workaround (all platforms):** install with `--host claude` (builds all binaries and creates the skill structure), then symlink the skill folders into Cursor's skills directory.

---

## 2. Prerequisites

| Requirement | Windows | macOS | Linux |
|---|---|---|---|
| Shell | Git Bash (required) | Terminal (bash/zsh) | Terminal (bash/zsh) |
| Git | git-scm.com/download/win | Xcode CLT: `xcode-select --install` | `sudo apt install git` / `sudo dnf install git` |
| Node.js LTS | nodejs.org installer | `brew install node` | `sudo apt install nodejs npm` / use `nvm` |
| Bun 1.0+ | `npm install -g bun` | `curl -fsSL https://bun.sh/install \| bash` | `curl -fsSL https://bun.sh/install \| bash` |

### Verify prerequisites (all platforms)

```bash
git --version
node --version
bun --version
```

### Platform-specific notes

- **Windows:** gstack must run from **Git Bash**, not PowerShell or CMD. Bun has a known Playwright bug on native Windows (bun#4253); the browse server falls back to Node.js automatically.
- **macOS:** gstack auto-installs `coreutils` via Homebrew during setup (for `gtimeout` hang protection in `/codex` and `/autoplan`). Set `GSTACK_SKIP_COREUTILS=1` to skip. Install Homebrew first: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
- **Linux:** No special caveats. `gtimeout` comes from `coreutils`, which is pre-installed on most distros.

---

## 3. Install — step by step

All commands below run in your terminal (**Git Bash on Windows**, Terminal on macOS/Linux).

### 3.1 Clone gstack (all platforms)

```bash
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/gstack
cd ~/gstack
```

### 3.2 Run setup with `--host claude` (all platforms)

```bash
./setup --host claude
```

This builds the `browse` binary, installs dependencies, and creates the standard skill layout at `~/.claude/skills/`.

### 3.3 Symlink gstack skills into Cursor

Cursor's skills directory is `~/.cursor/skills-cursor/` on all platforms. Run this loop in your terminal:

```bash
mkdir -p ~/.cursor/skills-cursor
for skill in ~/gstack/*/SKILL.md; do
  name=$(basename "$(dirname "$skill")")
  ln -sfn "$(dirname "$skill")" "$HOME/.cursor/skills-cursor/gstack-$name"
done
echo "Linked: $(ls ~/.cursor/skills-cursor/ | grep -c '^gstack-') gstack skills"
```

Expected output: `Linked: 30 gstack skills` (or similar).

> **If your Cursor skills folder is different on your machine**, find it with: `ls ~/.cursor/` — you're looking for a folder named `skills-cursor`, `skills`, or similar. Replace the path in the loop accordingly.

### 3.4 Verify

```bash
ls ~/.cursor/skills-cursor/ | grep gstack
```

You should see entries like `gstack-autoplan`, `gstack-office-hours`, `gstack-plan-eng-review`, etc.

### 3.5 Restart Cursor

- **Windows:** Close all Cursor windows (check tray icon too).
- **macOS:** `Cmd+Q` to fully quit (closing the window is not enough).
- **Linux:** Fully quit the application from the window menu.

Reopen Cursor — gstack skills now appear alongside existing ones (`babysit`, `create-rule`, etc.).

---

## 4. How to use gstack for planning

Cursor triggers skills based on the SKILL.md description matching your request. Use natural language:

| What you say to Cursor | Skill triggered | What it does |
|---|---|---|
| "Run gstack office hours on this idea: [describe]" | `gstack-office-hours` | 6 forcing questions, reframes the problem, writes design doc |
| "Use gstack autoplan to plan [feature]" | `gstack-autoplan` | Auto-runs CEO → eng → design → devex reviews |
| "Run gstack plan eng review on [doc]" | `gstack-plan-eng-review` | Locks architecture, edge cases, test matrix |
| "Run gstack plan devex review on the API" | `gstack-plan-devex-review` | DX review for API / CLI / SDK work |
| "Run gstack review on my changes" | `gstack-review` | Staff-engineer-level code review, auto-fixes obvious bugs |
| "Run gstack qa on [URL]" | `gstack-qa` | Opens a real browser, clicks through flows, reports bugs |
| "Run gstack ship" | `gstack-ship` | Sync main, run tests, push, open PR |

### Recommended planning workflow (applies to any project)

```
1. "Run gstack office hours: I want to add [feature]"
   → writes docs/plans/[feature].md

2. "Run gstack autoplan on docs/plans/[feature].md"
   → CEO + eng + devex review, produces reviewed plan

3. Approve the plan, exit plan mode, let Cursor implement it.

4. "Run gstack review on my diff"
   → catches bugs, auto-fixes obvious ones.

5. "Run gstack ship"
   → runs tests, opens PR.
```

---

## 5. Keeping gstack updated (all platforms)

```bash
cd ~/gstack && git pull
```

The Cursor symlinks point into `~/gstack/`, so `git pull` updates them automatically. If **new skill folders** are added upstream, re-run the symlink loop from **Step 3.3**.

---

## 6. Troubleshooting

### `./setup --host cursor` fails with "Unknown --host value" (all platforms)
Expected. Use `--host claude` instead.

### Bun not found after install

- **Windows:** Restart Git Bash. If still missing, reinstall with `npm install -g bun`.
- **macOS / Linux:** Add to shell profile: `echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.zshrc` (or `~/.bashrc`), then `source ~/.zshrc`.

### Playwright / browse errors

- **Windows:** Install Node.js; make sure both `node` and `bun` are on PATH. Browse server falls back to Node.js automatically.
- **macOS:** First-time Playwright may need: `bunx playwright install chromium`
- **Linux:** May need additional system libraries: `sudo apt install libnss3 libatk-bridge2.0-0 libgtk-3-0 libgbm1`

### macOS: setup complains about `coreutils`
Either install Homebrew and let setup auto-install, or skip it:
```bash
GSTACK_SKIP_COREUTILS=1 ./setup --host claude
```

### Cursor doesn't see the skills after restart

- Confirm symlinks exist: `ls -la ~/.cursor/skills-cursor/ | grep gstack`
- Confirm `SKILL.md` is readable: `cat ~/.cursor/skills-cursor/gstack-office-hours/SKILL.md | head -5`
- Fully quit Cursor (not just close window) before reopening.
- On Linux, if `~/.cursor/` doesn't exist, Cursor may store data elsewhere — check `~/.config/Cursor/` and create the symlinks there instead.

### Skill doesn't trigger from natural language
Cursor matches on the SKILL.md description, not the folder name. Use phrases that reflect the skill's purpose (e.g. "plan a feature", "review my code", "test this URL").

---

## 7. Uninstall

### Remove Cursor symlinks (all platforms)

```bash
find ~/.cursor/skills-cursor -maxdepth 1 -type l -name 'gstack-*' -delete
```

### Remove gstack itself (all platforms)

```bash
~/gstack/bin/gstack-uninstall
```

Or manually:

```bash
rm -rf ~/gstack
rm -rf ~/.claude/skills/gstack*
rm -rf ~/.gstack
```

### Platform-specific cleanup

- **macOS:** Playwright cache at `~/Library/Caches/ms-playwright/` is left in place (other tools may use it). Remove with `rm -rf ~/Library/Caches/ms-playwright` if nothing else needs it.
- **Linux:** Playwright cache at `~/.cache/ms-playwright/`. Remove similarly if unused.
- **Windows:** Playwright cache at `%USERPROFILE%\AppData\Local\ms-playwright\`. Remove via File Explorer or PowerShell.

---

## 8. Quick reference — skill index

**Planning skills (before you code):**
- `gstack-office-hours` — Start here. 6 forcing questions, reframes your idea.
- `gstack-plan-ceo-review` — CEO-level scope challenge.
- `gstack-plan-eng-review` — Architecture + diagrams + test matrix.
- `gstack-plan-design-review` — Design system review (UI work).
- `gstack-plan-devex-review` — DX review (API / CLI / SDK work).
- `gstack-autoplan` — Runs all the above in one command.

**Build / review skills (after you code):**
- `gstack-review` — Staff-engineer code review.
- `gstack-investigate` — Root-cause debugging.
- `gstack-codex` — Second opinion from OpenAI Codex CLI.
- `gstack-design-review` — Live design audit.
- `gstack-devex-review` — Live DX audit.
- `gstack-cso` — Security audit (OWASP + STRIDE).

**Ship skills:**
- `gstack-ship` — Run tests, open PR.
- `gstack-land-and-deploy` — Merge + deploy + verify.
- `gstack-canary` — Post-deploy monitoring.
- `gstack-document-release` — Update all docs to match shipped code.
- `gstack-retro` — Weekly engineering retro.

**QA / browser skills:**
- `gstack-qa` — Real-browser testing.
- `gstack-qa-only` — Test + report only (no code changes).
- `gstack-browse` — Generic browser automation.
- `gstack-setup-browser-cookies` — Import real browser cookies.

**Safety skills:**
- `gstack-careful` — Warn before destructive commands.
- `gstack-freeze` — Lock edits to one directory.
- `gstack-guard` — `careful` + `freeze` combined.
- `gstack-unfreeze` — Remove freeze.

**Meta:**
- `gstack-learn` — Review / prune what gstack learned about your codebase.
- `gstack-gstack-upgrade` — Self-update.

---

*Generated as a setup reference. Source: https://github.com/garrytan/gstack — MIT License.*
