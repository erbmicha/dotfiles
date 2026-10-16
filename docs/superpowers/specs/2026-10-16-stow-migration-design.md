# Dotfiles migration to GNU Stow

## Goal

Organize `/Users/erb/dotfiles` into independently installable Stow packages,
preserving the machine's current working configuration and all original files.

## Current state

- GNU Stow 2.4.1 is installed.
- The dotfiles directory has no Git repository yet.
- Home-directory symlinks point directly into the flat dotfiles directory.
- `~/.config` and `~/.tmux` link entire directories into dotfiles, including
  application state, authentication data, and downloaded plugins.
- The active `~/.zshrc` and `~/.bashrc` differ from the dotfiles copies.
- The active `~/Brewfile` also conflicts with the dotfiles copy.
- A Stow simulation reports conflicts for those three files and `.DS_Store`.

## Selected approach

Use one package per tool, with paths beneath each package mirroring `$HOME`:

```text
dotfiles/
  zsh/.zshrc
  zsh/.zshenv
  zsh/.zsh_functions
  zsh/.p10k.zsh
  bash/.bashrc
  shell/.profile
  git/.gitconfig
  git/.config/git/ignore
  ruby/.gemrc
  ruby/.rubocop.yml
  tmux/.tmux.conf
  ghostty/.config/ghostty/config
  mise/.config/mise/config.toml
  ...additional reviewed tool packages...
  Brewfile
  README.md
  .gitignore
  .stowrc
  docs/
```

Root documentation, the Brewfile, and repository metadata are not Stow packages.
Keep the Brewfile at the repository root and document its explicit use with
Homebrew; preserve the existing home Brewfile separately. Keep `.asdfrc` and
`.skylight` local unless review establishes they are still needed and safe to
manage. Package other preferences only after reviewing their file roles.

The alternative of retaining `stow .` requires fewer moves but keeps unrelated
tools coupled. A single package containing all configurations has the same
limitation. The selected approach allows tool-by-tool installation and removal.

## Preservation and migration

1. Record existing symlink targets and create a timestamped backup outside
   the repository before changing active files. Preserve file permissions.
2. Treat the active home shell files as authoritative. Preserve the divergent
   dotfiles versions in the backup; do not silently merge old settings back in.
3. Replace the whole-directory `.config` link with a real directory. Restore
   local state there and move only reviewed portable settings into packages.
   Use Stow's `--no-folding` behavior so applications can create adjacent local
   files without placing them in the repository.
4. Preserve downloaded tmux plugins locally under a real `~/.tmux` directory;
   manage reviewed configuration files separately. Inspect references before
   relocating anything that might be sourced by the shell or tmux.
5. Remove or replace only the old symlinks whose targets were recorded. Stow
   selected packages with an explicit home target. Never use `--adopt` to
   resolve divergent files automatically.
6. If any additional conflict appears, retain both versions and resolve it
   without overwriting the active configuration.

## Local state and sensitive configuration

Authentication files, databases, histories, logs, sessions, caches, temporary
files, and downloaded dependencies remain outside managed packages. Examples
already present include GitHub/Copilot authentication data, Swamp authentication,
FileZilla connection data, and editor session files.

Review shell and tool configuration for embedded secrets without printing their
values. If a portable file contains credentials, preserve the active file locally
and defer managing it until its secret-dependent behavior can be preserved safely.
Do not introduce a new secret-management dependency as part of this cleanup.

Only explicitly reviewed files may be staged in Git. Existing ignore patterns
must be updated for the package layout. Preserve useful dependency lockfiles
when they belong to a reviewed configuration; do not ignore all lockfiles.

## Documentation and verification

The README will list the actual packages and show commands for dry-run,
installation, restowing, and removal, with an explicit target and `--no-folding`.
It will explain local state, backups, and Brewfile usage.

Before changing live links, simulate installation into an isolated home target
and verify installation, repeat installation, and removal. Then simulate against
the actual home directory and resolve all conflicts. After migration, confirm:

- Every intended link resolves to the correct package file.
- Active file contents match their preserved originals except documented edits.
- Existing application state and tmux plugins remain accessible.
- Shell files pass syntax checks without executing interactive startup commands.
- Stow reports no conflicts and no pending changes when rerun.
- Git contains no credentials or runtime data.

## Decisions

- User: "i prefer your recommendations" — selects separate packages by tool,
  selective `.config` links, local runtime state, backups, and dry-run validation
  as proposed in the preceding response.
- Preserve current behavior and use active shell files as the starting point —
  avoids replacing newer working settings with divergent older copies.
- Keep this migration local — publishing a remote repository is outside scope.

## Scope

This change organizes existing settings for Stow. It does not redesign shell
behavior, uninstall software, delete old settings, refresh all dependencies, or
publish configuration. Implementation begins after review of this written design.
