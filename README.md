# My dotfiles for work at [Sun*](https://sun-asterisk.com/)

PhDoanh's dotfiles managed by [chezmoi](https://www.chezmoi.io/) - targeting macOS and Ubuntu Linux (includes WSL2), with Sun* work-machine awareness and Docker support.

## 📦 Components

| Component | chezmoi Source File | Destination | Description |
|-----------|-------------------|-------------|-------------|
| **Zsh config** | `dot_zshrc.tmpl` | `~/.zshrc` | Shell entrypoint; loads Oh My Zsh, Powerlevel10k, rbenv, opencode, aliases & secrets |
| **Zsh aliases** | `dot_zsh_aliases.tmpl` | `~/.zsh_aliases` | Navigation helpers, Docker/Compose shortcuts, universal package manager (`p`), search & grep utils |
| **Zsh secrets** | `encrypted_dot_zsh_secrets.age` | `~/.zsh_secrets` | Age-encrypted secrets file (sourced in `.zshrc`) |
| **Powerlevel10k config** | `dot_p10k.zsh` | `~/.p10k.zsh` | Full Powerlevel10k prompt configuration |
| **Git config** | `dot_gitconfig.tmpl` | `~/.gitconfig` | Global git config with conditional include for personal/work identity |
| **Git config – personal** | `create_dot_gitconfig-personal.tmpl` | `~/.gitconfig-personal` | Personal git identity override (applied in `~/projects/`) |
| **Git config – work** | `create_dot_gitconfig-work.tmpl` | `~/.gitconfig-work` | Sun* work git identity (applied in `~/projects/sun-asterisk/`) |
| **Gem config** | `dot_gemrc` | `~/.gemrc` | RubyGems global settings |
| **chezmoi config template** | `.chezmoi.toml.tmpl` | `~/.config/chezmoi/chezmoi.toml` | chezmoi config: age encryption, VSCode as editor/diff/merge tool, auto-commit & push |
| **External resources** | `.chezmoiexternal.toml.tmpl` | _(managed by chezmoi)_ | Downloads Oh My Zsh, Powerlevel10k, zsh plugins, MesloLGS NF fonts; also `gh`, `uv`, `opencode`, `lazygit` on Linux |
| **Age private key (encrypted)** | `key.txt.age` | `~/.config/chezmoi/key.txt` _(decrypted)_ | Age private key used to decrypt secrets; itself encrypted with a passphrase |
| **Oh My Zsh** | _(external)_ | `~/.oh-my-zsh` | Shell framework, pulled from GitHub archive |
| **zsh-syntax-highlighting** | _(external)_ | `~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting` | Syntax highlighting plugin for Zsh |
| **zsh-autosuggestions** | _(external)_ | `~/.oh-my-zsh/custom/plugins/zsh-autosuggestions` | Fish-like autosuggestions for Zsh |
| **Powerlevel10k theme** | _(external)_ | `~/.oh-my-zsh/custom/themes/powerlevel10k` | Fast, feature-rich Zsh theme |
| **MesloLGS NF fonts** | _(external, non-headless only)_ | `~/.local/share/fonts/` (Linux) / `~/Library/Fonts/` (macOS) | Nerd Font required for Powerlevel10k icons |
| **gh CLI** | _(external, Linux only)_ | `~/.local/bin/gh` | GitHub CLI (latest release) |
| **uv** | _(external, Linux only)_ | `~/.local/bin/uv` | Extremely fast Python package manager |
| **opencode** | _(external, Linux only)_ | `~/.opencode/bin/opencode` | AI coding agent CLI |
| **lazygit** | _(external, Linux only)_ | `~/.local/bin/lazygit` | Terminal UI for git |

### Data files (`.chezmoidata/`)

| File | Purpose |
|------|---------|
| `packages.toml` | System package lists for Linux (`apt`) and macOS (`brew`) — system, Rails build deps, Node.js, MySQL |
| `versions.toml` | Pinned versions: Ruby `3.3.11`, Rails `8.1.3` |
| `public-pii.toml` | Non-secret personal/work email addresses used in git config templates |

### Setup scripts (`.chezmoiscripts/`, run in order on change)

| Script | When | Action |
|--------|------|--------|
| `run_onchange_before_00-decrypt-private-key.sh.tmpl` | Before apply | Decrypts `key.txt.age` → `~/.config/chezmoi/key.txt` using a passphrase |
| `run_onchange_before_10-install-sys-packages.sh.tmpl` | Before apply | Installs system packages via `apt-get` / `brew` |
| `run_onchange_before_20-install-rbenv.sh.tmpl` | Before apply | Installs rbenv |
| `run_onchange_before_30-install-ruby.sh.tmpl` | Before apply | Installs Ruby `3.3.11` via rbenv |
| `run_onchange_before_40-install-rails.sh.tmpl` | Before apply | Installs Rails `8.1.3` and build dependencies |
| `run_onchange_before_50-install-nodejs-yarn.sh.tmpl` | Before apply | Installs Node.js + Yarn via nvm or brew |
| `run_onchange_before_60-install-mysql.sh.tmpl` | Before apply | Installs MySQL server/client |
| `run_onchange_before_70-install-vscode.sh.tmpl` | Before apply | Installs VS Code (full GUI on non-headless, CLI-only on headless/ephemeral) |

## ✅ Prerequisites

Before running the initial setup, ensure the following are available on your machine:

- **Ubuntu Linux or macOS**: Scripts are conditioned on `linux-ubuntu` and `darwin`; other distros are not supported
- **Docker**: Required only if you want to use the Docker-based setup ([Option B](#option-b---docker-chezmoi-native)). Ensure Docker is installed and running.
- **WSL2**: Required only if you are Windows user - install via `wsl --install` (all below operations are same to linux-ubuntu)

## 🚀 Initial Setup

### Option A - Native (Linux / macOS)

**Step 1: Install chezmoi and apply dotfiles in one command**

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply PhDoanh/dotfiles
```

This will:
1. Download and install chezmoi to `~/.local/bin`
2. Clone this repo to `~/.local/share/chezmoi`
3. Prompt you: _"Is this a Sun\* work machine?"_ — answer `y` or `n`
4. Run all `run_onchange_before_*` scripts in order (decrypt key → install packages → rbenv → Ruby → Rails → Node.js → MySQL → VSCode)
5. Apply all dotfiles to your home directory

**Step 2: Reload the shell**

```sh
exec zsh # or `chsh -s $(which zsh)` to set Zsh as default shell
```

> **Note:** The age passphrase for `key.txt.age` will be prompted during step 4. Keep it ready.

### Option B - Docker (chezmoi native)

chezmoi provides native `docker run` subcommands. That pulls a base image, creates a container, installs chezmoi inside it, runs `chezmoi init --apply`, then drops you into a shell — all in one command.

```sh
chezmoi docker run -p apt-get ubuntu:latest PhDoanh/dotfiles
```

> ⚠️ On Docker (ephemeral/headless mode), the scripts automatically detect `CODESPACES`, `REMOTE_CONTAINERS_IPC`, or `root`/`ubuntu`/`vscode` usernames and set `isEphemeral=true` and `isHeadless=true`. This means:
> - VS Code is installed in **CLI-only** mode (not full GUI)
> - MesloLGS NF fonts are **not** installed

