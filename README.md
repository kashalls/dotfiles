# dotfiles

My dotfiles managed by [mise](https://mise.jdx.dev/dotfiles.html).

## Layout

- `mise.toml` - shared config: `[dotfiles]` entries, git identity vars, and
  the nanorc syntax repo. Symlinked to `~/.config/mise/config.toml` on apply,
  so it is also the global mise config.
- `mise.<machine>.toml` - per-machine overlays: machine-only dotfiles, package
  hooks, and the `machine` template var. Selected via `MISE_ENV`.
- `home/` - the dotfile sources. Plain files are symlinked into `$HOME`;
  `*.tmpl` files are rendered with the mise (Tera) template engine
  (`~/.gitconfig` and the agent instructions).

## Machines

| Machine   | Envs (`miserc.toml`)  | Shell      | Packages |
| --------- | --------------------- | ---------- | -------- |
| `wsl`     | `["linux", "wsl"]`    | bash       | Homebrew |
| `linux`   | `["linux"]`           | bash       | Homebrew |
| `macos`   | `["macos"]`           | zsh        | Homebrew |

`wsl` is layered on top of `linux`: it reuses the Linux Brewfile and hook and
only overrides `machine`, which the templates use for WSL-specific bits
(`ssh.exe`, 1Password `op-ssh-sign-wsl.exe`).

The machine is pinned once per host in `~/.config/mise/miserc.toml` (untracked):

```sh
mkdir -p ~/.config/mise
printf 'env = ["linux", "wsl"]\n' > ~/.config/mise/miserc.toml
```

Git identity defaults live in `[vars]` in `mise.toml`; override them per host
in an untracked `mise.local.toml` next to `mise.toml`.

## Bootstrap

### Linux, WSL, macOS

Install Homebrew first if you want the Brewfile applied; the post-dotfiles hook
skips `brew bundle` on Linux when brew is missing.

```sh
curl https://mise.run | sh   # installs ~/.local/bin/mise
git clone https://github.com/kashalls/dotfiles ~/.dotfiles
cd ~/.dotfiles
mise trust
mkdir -p ~/.config/mise
printf 'env = ["linux", "wsl"]\n' > ~/.config/mise/miserc.toml   # see Machines
mise bootstrap --yes
```

## Notes

- Re-run `mise bootstrap --yes` (or `mise dotfiles apply`) after pulling.
  The `post-dotfiles` hook (`brew bundle`) runs on both.
- Applying replaces existing unmanaged targets such as `~/.gitconfig`; back
  them up first on a new machine.
- Symlinked files edit the repo checkout directly. Rendered templates need
  `mise dotfiles apply` after editing their `.tmpl` source.
- `~/.claude/CLAUDE.md`, `~/.codex/AGENTS.md` and `~/.config/agents/AGENTS.md`
  are all rendered from `home/config/agents/AGENTS.md.tmpl`.
- `mise dotfiles status` shows what is applied, missing, or drifted.
