# dotfiles

My shell, editor, and CLI tool setup, managed with [chezmoi](https://www.chezmoi.io/) and [mise](https://mise.jdx.dev/). The same repo sets up my Arch desktop, a Mac or WSL machine, and DevPod dev containers.

## Part of the stateless workstation

This repo is one of three that set up my environment from scratch:

- dotfiles (this repo): shell, editor, and CLI tools, used on Arch, macOS, WSL, and inside containers
- [dev-container-playground](https://github.com/MechHead777/dev-container-playground): project environments with DevPod, which applies this repo inside each container
- arch-bootstrap (planned): Arch install, pacman packages, system services, then this repo and DevPod setup

## What's Included

| Area | Files |
|---|---|
| Shells | `.zshrc`, `.bashrc` (mise activation, starship, zoxide, fzf, aliases) |
| CLI tools | `.config/mise/config.toml` (installed per user by mise) |
| Prompt | `.config/starship/config.toml` (two-line prompt) |
| Editor | `.config/nvim` (LazyVim, with plugin versions pinned in `lazy-lock.json`) |
| Terminal | `.config/kitty`, `.config/tmux`, `.config/yazi`, `.config/bat` |
| Desktop (Arch only) | `.config/niri`, `.config/fuzzel`, `~/.local/bin/cliphist-fuzzel-img` |

# Install

**Warning: applying this repo replaces existing files such as `~/.zshrc` and `~/.bashrc`. Back up anything you want to keep before running `chezmoi init --apply`.**

Your terminal needs a [Nerd Font](https://www.nerdfonts.com/) for the eza and starship icons to show up.

## Arch

First, install git and mise: `sudo pacman -S --needed git mise`

Then, apply the dotfiles, using mise to run chezmoi once:

```sh
mise exec chezmoi -- chezmoi init --apply MechHead777/dotfiles
```

After it's finished, start a new shell so mise's tools are on your PATH: `exec zsh`

> nvim-treesitter builds its parsers with a C compiler, so install `base-devel` if it isn't already there.

## macOS, WSL, or Another Linux

First, install chezmoi to `~/.local/bin` and apply the dotfiles in one step:

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- -b "$HOME/.local/bin" init --apply MechHead777/dotfiles
```

If mise isn't installed, chezmoi downloads it to `~/.local/bin` before installing the tools.

After it's finished, start a new shell: `exec zsh`

> On macOS, mise has no prebuilt eza download, so it falls back to building it. If that fails, it's often because Rust isn't installed, and `brew install eza` works instead.

## DevPod

Point DevPod at this repo and its setup script:

```sh
devpod context set-options \
  -o DOTFILES_URL=https://github.com/MechHead777/dotfiles \
  -o DOTFILES_SCRIPT=setup
```

New workspaces clone this repo into `~/dotfiles` and run `setup`, which applies it with chezmoi without asking any questions.

> DevPod clones the dotfiles once and never pulls. If a workspace has old dotfiles, delete it and create a new one.

# How It Works

| Path | Purpose |
|---|---|
| `dot_*`, `private_*`, `executable_*` | chezmoi source names. `dot_zshrc` becomes `~/.zshrc`, `private_` sets owner-only permissions, and `executable_` makes the file executable |
| `.chezmoiexternals/mise.toml` | Downloads mise to `~/.local/bin`, but only when mise isn't already on the PATH |
| `.chezmoiscripts/run_onchange_after_mise-install.sh.tmpl` | Runs `mise install` after the files are applied, and runs again whenever `config.toml` changes |
| `.chezmoiignore` | Keeps `README.md` and `setup` out of `~`, and skips the desktop files on macOS, WSL, and in containers |
| `setup` | DevPod entry point. It points chezmoi at DevPod's clone, installs chezmoi if needed, and applies the repo |

# Making Changes

Edit the source copy, not the file in your home directory, then apply it:

```sh
chezmoi edit ~/.zshrc
chezmoi apply
```

Some files are changed by tools instead of by hand. Copy those changes back into the source after running:

- `mise use -g <tool>` changes `~/.config/mise/config.toml`
- `:Lazy update` in nvim changes `~/.config/nvim/lazy-lock.json`

```sh
chezmoi re-add
chezmoi cd
git commit -am "chore: sync tool changes"
git push
```

> If `chezmoi apply` asks whether to overwrite a file, it's often because that file was edited directly. Choose `diff` first, and use `chezmoi re-add` if the home copy is the one to keep.

To pull the latest changes onto another machine, run `chezmoi update`.

# Design Decisions

- **CLI tools come from mise, not pacman.** The same `config.toml` works on Arch, macOS, WSL, and in containers without root. pacman keeps the OS, desktop, and system services.
- **There is no global `mise.lock`.** Tools track `latest`. Keeping a lockfile in sync added upkeep that wasn't worth it for everyday CLI tools. Project tools such as kubectl and helm are pinned in each project's own `mise.toml` instead.
- **The LazyVim starter is committed, not downloaded.** Committing it keeps `lazy-lock.json` in the repo, so plugin versions match on every machine.
- **The repo is public and cloned over HTTPS.** Containers can apply it without SSH access. Secrets never go in this repo.
