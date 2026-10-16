# My dotfiles

This directory contains my dotfiles. I use [GNU Stow](https://www.gnu.org/software/stow/) to manage them.

## Requirements

Ensure you have the following installed on your system:

### Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Git

```bash
brew install git
```

### Stow

```bash
brew install stow
```

## Installation

First, check out the dotfiles repo in your $HOME directory using git:

```bash
cd ~
git clone git@github.com/erbmicha/dotfiles.git
cd dotfiles
```

then use GNU Stow to create the symlinks:

```bash
stow .
```
