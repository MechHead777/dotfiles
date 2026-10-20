# dotfiles

Shell, editor, and CLI configuration managed with [chezmoi](https://www.chezmoi.io/) and [mise](https://mise.jdx.dev/). The same repo supports Arch, macOS, WSL, and DevPod containers.

## Part of the stateless workstation

The workstation setup is split into three repos, used in this order:

- [arch-bootstrap](https://github.com/MechHead777/arch-bootstrap): OS layer: Arch installation, system packages, and services.
- dotfiles (this repo): User layer: shell, editor, and CLI tools with chezmoi and mise.
- [dev-container-playground](https://github.com/MechHead777/dev-container-playground): Project layer: per-project environments with DevPod and Docker.

## What's Included

| Area | Configuration |
|---|---|
| Shells | `.zshrc` and `.bashrc`: mise activation, starship, zoxide, fzf, and aliases |
| CLI tools | `.config/mise/config.toml`: per-user tools, including GitHub CLI |
| Prompt | `.config/starship/config.toml`: two-line prompt |
| Editor | `.config/nvim`: LazyVim with plugin versions pinned in `lazy-lock.json` |
| Terminal tools | Kitty, tmux, yazi, and bat configuration |
| Desktop | niri, fuzzel, and the clipboard-history script; skipped on macOS, WSL, and in containers |

## Setup

**Warning: applying this repo replaces existing files such as `~/.zshrc` and `~/.bashrc`. Back up anything you want to keep before running `chezmoi init --apply`.**

Use a [Nerd Font](https://www.nerdfonts.com/) in the terminal for eza and starship icons.

### Arch

For a full workstation setup, follow [arch-bootstrap](https://github.com/MechHead777/arch-bootstrap#setup). It installs the system packages, applies this repo, and configures DevPod. To apply only the dotfiles to an existing Arch installation, use the steps below.

First, install git and mise:

```sh
sudo pacman -S --needed git mise
```

Then, apply the dotfiles, using mise to run chezmoi:

```sh
mise exec chezmoi -- chezmoi init --apply MechHead777/dotfiles
```

> nvim-treesitter builds its parsers with a C compiler. Install `base-devel` if it isn't already available.

After it's finished, start a new shell so the tools are on your PATH:

```sh
exec zsh
```

### macOS, WSL, or another Linux

First, install chezmoi to `~/.local/bin` and apply the dotfiles:

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- -b "$HOME/.local/bin" init --apply MechHead777/dotfiles
```

If mise isn't installed, chezmoi downloads it to `~/.local/bin` before installing the tools.

> On macOS, mise has no prebuilt eza download, so it falls back to building it. If that fails, it's often because Rust isn't installed, and `brew install eza` works instead.

After it's finished, start a new shell:

```sh
exec zsh
```

### DevPod

The Arch bootstrap configures these dotfiles options. For other machines, or a manual setup, point DevPod at this repo and its setup script:

```sh
devpod context set-options \
  -o DOTFILES_URL=https://github.com/MechHead777/dotfiles \
  -o DOTFILES_SCRIPT=setup
```

New workspaces clone this repo into `~/dotfiles` and run `setup`, which applies it with chezmoi without asking any questions. Follow [dev-container-playground](https://github.com/MechHead777/dev-container-playground#setup) for the full host and workspace setup.

> DevPod clones the dotfiles once and never pulls. To pick up newer dotfiles in a workspace, recreate it as described in the workspace guide.

## Usage

### Edit configuration

Edit the source copy, then apply it:

```sh
chezmoi edit ~/.zshrc
chezmoi apply
```

> If chezmoi asks whether to overwrite a file, choose `diff` first. Use `chezmoi re-add` if the home copy is the one to keep.

### Save changes made by tools

Some tools edit the home copy directly:

| Command | File changed |
|---|---|
| `mise use -g <tool>` | `~/.config/mise/config.toml` |
| `:Lazy update` in nvim | `~/.config/nvim/lazy-lock.json` |

Copy those changes back into the source, review the diff, and commit:

```sh
chezmoi re-add
chezmoi cd
git diff
git commit -am "chore: sync tool changes"
git push
```

### Update another machine

Pull and apply the latest committed changes:

```sh
chezmoi update
```

## Verification

After setup, check the managed files and tool paths:

```sh
chezmoi status
command -v bat eza starship gh
```

With no local changes, `chezmoi status` should print nothing. The tool commands should resolve through mise. Start a new shell if they aren't on your PATH yet.

## Repository Layout

| Path | Purpose |
|---|---|
| `dot_*`, `private_*`, `executable_*` | chezmoi source names: `dot_` creates dotfiles, `private_` sets owner-only permissions, and `executable_` makes scripts executable |
| `dot_config/mise/config.toml` | Global CLI tool selection |
| `.chezmoiexternals/mise.toml` | Downloads mise to `~/.local/bin` when it isn't already on the PATH |
| `.chezmoiscripts/run_onchange_after_mise-install.sh.tmpl` | Runs `mise install` after files are applied and whenever the tool configuration changes |
| `.chezmoiignore` | Keeps repo documentation out of `~` and skips desktop files on macOS, WSL, and in containers |
| `setup` | DevPod entry point: points chezmoi at the clone, installs chezmoi if needed, and applies the repo |

## Design Decisions

| Decision | Choice | Reasoning |
|---|---|---|
| CLI tools | mise | The same configuration works across Arch, macOS, WSL, and containers without root. System and desktop packages stay in the OS package manager |
| Global tool versions | No global `mise.lock` | Everyday CLI tools track `latest`; project tools such as kubectl and helm are pinned in each project's `mise.toml` |
| Editor configuration | Committed LazyVim starter | Keeps `lazy-lock.json` in the repo so plugin versions match across machines |
| Repository access | Public HTTPS clone | Containers can apply the dotfiles without SSH access. Secrets stay outside the repo |
