# Cross-machine Tech Lead configuration (macOS + WSL2 Ubuntu)

This repo is a **configuration source**, not a shared live Hermes profile.
The two machines run their own local Hermes, Herdr Core, OMP, sessions, credentials and worktrees. Only versioned instructions/templates are shared.

## What lives where

- `distribution.yaml`, `SOUL.md`, `config.yaml`, `skills/`: Hermes Profile Distribution payload. The upstream official `herdr-core` skill is installed separately and not overwritten by this curated repo.
- `host/lead`: simple Bash command entry point. A machine may have only one active Hermes process with profile `tech-lead` at a time; project-specific *conversation* titles are NOT separate profiles.
- `host/omp-worker.md`: Herdr Core template, used to start native interactive OMP.
- `.env`, `auth.json`, `sessions/`, `memories/`, provider login state, local Git checkouts and worktrees: **do not put these in Git**.

## Create the private configuration repository on WSL2

Unpack this ZIP to `~/agent-config` on WSL2 and compare the included `SOUL.md` and custom skill with your currently tested versions **before replacing anything**. Incorporate any personal changes into the source copy. Then initialize a private Git repository:

```bash
cd ~/agent-config
git init -b main
git add .gitignore README.md distribution.yaml config.yaml SOUL.md \
  skills/herdr-spec-execution/SKILL.md host/lead host/omp-worker.md
git diff --cached --name-only   # inspect before commit
git commit -m "Initial portable tech lead configuration"
git remote add origin git@github.com:YOUR_NAME/agent-config.git
git push -u origin main
```

Create the remote repository privately first. Never stage the **live** `~/.hermes/profiles/tech-lead` tree or any credentials, local memories, sessions or worktrees.

## First use on Mac (or another machine without this profile)

1. Install `hermes`, `hc` (Herdr Core), `omp`, `git` and check `command -v` for each. Mac must have its own model/provider auth.
2. Clone this private repository into a local folder, e.g. `~/agent-config`.
3. Inspect files and install the Hermes distribution (use its actual local folder):

```bash
hermes profile install "$HOME/agent-config" --name tech-lead
hermes -p tech-lead setup                   # configure credentials/model locally if needed
hermes -p tech-lead config check
```

4. Install the official Herdr Core skill into this local Hermes profile (the repo pins the reviewed upstream revision):

```bash
hermes -p tech-lead skills install "https://raw.githubusercontent.com/herdrdev/herdr-core/c7517808d64f06609e4ecf0db1757005373cc3a3/skills/herdr-core/SKILL.md"
```

5. Install the two host entry points (ensure PATH contains ~/.local/bin):

```bash
mkdir -p "$HOME/.local/bin" "${XDG_CONFIG_HOME:-$HOME/.config}/herdr-core/templates"
install -m 755 "$HOME/agent-config/host/lead" "$HOME/.local/bin/lead"
install -m 644 "$HOME/agent-config/host/omp-worker.md" "${XDG_CONFIG_HOME:-$HOME/.config}/herdr-core/templates/omp-worker.md"
hc templates
```

6. In any local repository, `lead`. The user writes natural-language tasks; Hermes chooses one or many native OMP workers. Run `hc --skill` to inspect runtime behavior as installed.

## Existing WSL2 installation (keep current state)

DO NOT run `hermes profile install ... --name tech-lead` on an already-existing profile or delete the existing profile. Back up and apply **only authored, non-sensitive files** if you intend to keep the same profile:

```bash
SRC="$HOME/agent-config"
LEAD_HOME="$HOME/.hermes/profiles/tech-lead"
BACKUP="$HOME/agent-config-backups/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP" "$LEAD_HOME/skills/herdr-spec-execution"
cp -p "$LEAD_HOME/SOUL.md" "$BACKUP/SOUL.md"
[ ! -f "$LEAD_HOME/skills/herdr-spec-execution/SKILL.md" ] || cp -p "$LEAD_HOME/skills/herdr-spec-execution/SKILL.md" "$BACKUP/herdr-spec-execution.SKILL.md"
cp "$SRC/SOUL.md" "$LEAD_HOME/SOUL.md"
cp "$SRC/skills/herdr-spec-execution/SKILL.md" "$LEAD_HOME/skills/herdr-spec-execution/SKILL.md"
mkdir -p "$HOME/.local/bin" "${XDG_CONFIG_HOME:-$HOME/.config}/herdr-core/templates"
install -m 755 "$SRC/host/lead" "$HOME/.local/bin/lead"
install -m 644 "$SRC/host/omp-worker.md" "${XDG_CONFIG_HOME:-$HOME/.config}/herdr-core/templates/omp-worker.md"
```

Review SOUL differences before copying if you have added personal rules. Never copy the portable `config.yaml` over existing WSL2 local config: use `hermes -p tech-lead config get terminal.cwd` and `config check` to inspect. The WSL profile is NOT distribution-managed yet; after `git pull`, rerun only the authored-file copy section above to apply updates, or migrate separately to a fresh distribution-managed profile after safely preserving your state.

## Updating

- Maintain one private Git repository as the authoritative config source, commit reviewed modifications once, push.
- On a **distribution-managed** machine: `hermes profile update tech-lead` (keeps sessions, memories, credentials; local config is preserved by default), then `git pull` the host repo and re-install `host/lead` and `host/omp-worker.md` if changed. When installed from a local Git checkout, use the source command behavior supported by your Hermes version; alternatively install from your GitHub URL.
- On the existing **non-distribution-managed WSL** profile: `git pull`, inspect differences, and copy only `SOUL.md` and `skills/` from the source. Do not delete or reinstall a profile to synchronize.

## Remote access versus local execution

`lead` on the Mac uses local Mac CPU, files, Hermes/OMP credentials and `hc` terminals. To inspect/control WSL-hosted work from Mac, add the WSL2 host via Herdr Core `hc machine add` over authenticated SSH (Tailscale may provide connectivity). Use a stable SSH host alias and run `hc machine add --help`; do not forward the raw hc Unix socket. This remote option shares the server's active terminals, **not** Mac's local code workspace or local Hermes memory.

## Safety checks

- `git status --short` and `git diff --cached` before pushing; never commit credentials or conversation artifacts.
- `bash -n host/lead`
- `hermes -p tech-lead config check`
- `hermes -p tech-lead skills list`
- `hc templates`
- Confirm `lead` from repo A/B uses distinct titled conversations and workers have correct local cwd.
- The example `host/lead` uses Unix Bash, `hc`, `hermes`; it is designed for macOS and Linux, but the user's installed versions have not been tested by this answer.
