# shallow-backup

[![Downloads](http://pepy.tech/badge/shallow-backup)](http://pepy.tech/count/shallow-backup)
[![Build Status](https://travis-ci.com/alichtman/shallow-backup.svg?branch=master)](https://travis-ci.com/alichtman/shallow-backup)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/1719da4d7df5455d8dbb4340c428f851)](https://www.codacy.com/app/alichtman/shallow-backup?utm_source=github.com&utm_medium=referral&utm_content=alichtman/shallow-backup&utm_campaign=Badge_Grade)

<!-- [![Coverage Status](https://coveralls.io/repos/github/alichtman/shallow-backup/badge.svg?branch=master)](https://coveralls.io/github/alichtman/shallow-backup?branch=master) -->

`shallow-backup` lets you easily create lightweight backups of installed packages, applications, fonts and dotfiles, and automatically push them to a remote Git repository.

I use it to manage [my dotfiles](https://github.com/alichtman/dotfiles).

![Shallow Backup GIF Demo](img/shallow-backup-demo.gif)

# Contents

- [Why?](#why)
- [Installation](#installation)
  - [Method 1: <a href="https://pypi.org/project/shallow-backup/" rel="nofollow">pipx</a>](#method-1-pipx)
  - [Method 2: Install From Source](#method-2-install-from-source)
- [Dependencies](#dependencies)
- [Usage](#usage)
- [Recipes](#recipes)
  - [Maintain a separate repo for your dotfiles](#maintain-a-separate-repo-for-your-dotfiles)
  - [Synchronize dotfiles on multiple computers](#synchronize-dotfiles-on-multiple-computers)
  - [Reinstall dotfiles from a backup](#reinstall-dotfiles-from-a-backup)
- [What can I back up?](#what-can-i-back-up)
- [Configuration](#configuration)
  - [Conditional Backup and Reinstallation](#conditional-backup-and-reinstallation)
- [Git Integration](#git-integration)
  - [A Word of Caution](#a-word-of-caution)
  - [.gitignore](#gitignore)
- [Output Structure](#output-structure)
- [Want to Contribute?](#want-to-contribute)

### Why?

I wanted a tool that allows you to:

- Back up dotfiles _from where they live on the system_.
- Back up files from _any_ path on the system, not just `$HOME`.
- Reinstall them from the backup directory idempotently.
- Backup and reinstall files conditionally, so you can easily manage dotfiles across multiple systems.
- Copy files on installation and backup, as opposed to symlinking them.
- Backup package installations in a highly compressed manner.
- Not worry about accidentally doing something dangerous / destructive (is user-protective).

`shallow-backup` checks all of those boxes.

### Installation

---

> **Warning**
> Be careful running this with elevated privileges. Code execution can be achieved with write permissions on the config file.

#### Method 1: [`pipx`](https://pypi.org/project/shallow-backup/)

```bash
$ pipx install shallow-backup
```

#### Method 2: Install From Source

```bash
$ git clone https://www.github.com/alichtman/shallow-backup.git
$ cd shallow-backup
$ pip3 install .
```

### Dependencies

---

- `pre-commit`
- `trufflehog`

If you are missing the dependencies, you will be guided to install them.

### Usage

---

- To start the interactive program, run `$ shallow-backup`.
- To backup your dotfiles, run `$ shallow-backup --backup-dots`.

`shallow-backup` was built with scripting in mind. Every feature that's supported in the interactive program is supported with command line arguments.

```shell
Usage: shallow-backup [OPTIONS]

  Easily back up installed packages, dotfiles, and more.
  You can edit which files are backed up in ~/.shallow-backup.

  Written by Aaron Lichtman (@alichtman).

Options:

  --add-dot TEXT               Add a dotfile or dotfolder to config by path.
  --backup-all                 Full back up.
  --backup-configs             Back up app config files.
  --backup-dots                Back up dotfiles.
  --backup-fonts               Back up installed fonts.
  --backup-packages            Back up package libraries.
  --delete-config              Delete config file.
  --destroy-backup             Delete backup directory.
  --dry-run                    Don't backup or reinstall any files, just give
                               verbose output.

  --new-path TEXT              Input a new back up directory path.
  --no-new-backup-path-prompt  Skip setting new back up directory path prompt.
  --no-splash                  Don't display splash screen.
  --reinstall-all              Full reinstallation.
  --reinstall-configs          Reinstall configs.
  --reinstall-dots             Reinstall dotfiles and dotfolders.
  --reinstall-fonts            Reinstall fonts.
  --reinstall-packages         Reinstall packages.
  --remote TEXT                Set remote URL for the git repo.

  --edit                       Open config file in $EDITOR.
  -v, --version                Display version and author info.
  -h, -help, --help            Show this message and exit.
```

### Recipes

---

#### Maintain a separate repo for your dotfiles

`shallow-backup` makes this easy! After making your first backup, `cd` into the `dotfiles/` directory and run `$ git init`. Create a `.gitignore`, and a create / set up (link the upstream remote, etc) a new repo on your favorite version control platform. With operations involving the parent `shallow-backup` repo, `shallow-backup` will prompt you interactively to update the nested submodule. After that is taken care of, `shallow-backup` will move on to updating the parent. The `dotfiles` repo will be tracked as a submodule.

#### Synchronize dotfiles on multiple computers

Run `shallow-backup --backup-dots` on the first computer. Make a commit and push to the remote. Then pull these changes down on the second computer. Run `shallow-backup --backup-dots` on the second computer, and resolve the merge conflicts. Once you have a final version you're happy with, make a commit, push it, and run `shallow-backup --reinstall-dots`. On the first computer, pull the changes and run `shallow-backup --reinstall-dots`. Your changes are now sync'd across both computers and the remote repository.

#### Reinstall dotfiles from a backup

To reinstall your dotfiles, clone your dotfiles repo and make sure your shallow-backup config path can be found at either `~/.config/shallow-backup.conf` or `$XDG_CONFIG_HOME/.shallow_backup.conf`. Set the `backup-path` key in the config to the path of your cloned dotfiles. Then run `$ shallow-backup --reinstall-dots`.

When reinstalling your dotfiles, the top level `.git/`, `.gitignore`, `img/` and `README.md` files and directories are ignored.

### `zsh` Completions

Available in [`zsh-users/zsh-completions`](https://github.com/zsh-users/zsh-completions/blob/master/src/_shallow-backup). Follow the [installation instructions here](https://github.com/zsh-users/zsh-completions/tree/master#using-zsh-frameworks).

### What can I back up?

---

By default, `shallow-backup` backs these up.

1. Dotfiles and dotfolders

   - `.bashrc`
   - `.bash_profile`
   - `.gitconfig`
   - `.pypirc`
   - `.config/shallow-backup.json`
   - `.ssh/`
   - `.vim/`
   - `.zshrc`

2. App Config Files

   - VSCode
   - Sublime Text 2/3
   - Terminal.app

3. Installed Packages

   - `brew` and `cask`
   - `cargo`
   - `gem`
   - `pip`
   - `pip3`
   - `npm`
   - `macports`
   - `VSCode` Extensions
   - `Sublime Text 2/3` Packages
   - System Applications

4. User installed `fonts`.

### Configuration

If you'd like to modify which files are backed up, you can edit the `JSON` config file. This file is looked for in the following locations, in this order:

1. `$SHALLOW_BACKUP_CONFIG_DIR/shallow-backup.json`
2. `$XDG_CONFIG_HOME/shallow-backup.json`
3. `~/.config/shallow-backup.json`

#### Conditional Backup and Reinstallation

> **Warning**
> This feature allows code execution (by design). If untrusted users can write to your config, they can achieve code execution next time you invoke `shallow-backup` _backup_ or _reinstall_ functions. Starting in `v5.2`, the config file will have default permissions of `644`, and a warning will be printed if others can write to the config.

Every key under dotfiles has two optional subkeys: `backup_condition` and `reinstall_condition`. Both of these accept expressions that will be evaluated with `bash`. An empty string (`""`) is the default value, and is considered to be `True`. If the return value of the expression is `0`, this is considered `True`. Otherwise, it is `False`. This lets you do simple things like preventing backup with:

```javascript
// Because `$ false` returns 1
"backup_condition": "false"
```

And also more complicated things like only backing up certain files if an environment variable is set:

```javascript
"backup_condition": "[[ -n \"$ENV_VAR\" ]]"
```

Here's an example config based on my [dotfiles](https://www.github.com/alichtman/dotfiles):

```json
{
	"backup_path": "~/shallow-backup",
	"lowest_supported_version": "5.0.0a",
	"dotfiles": {
		".config/agignore": {
			"backup_condition": "uname -a | grep Darwin",
			"reinstall_condition": "uname -a | grep Darwin"
		},
		".config/git/gitignore_global": { },
		".config/jrnl/jrnl.yaml": { },
		".config/kitty": { },
		".config/nvim": { },
		".config/pycodestyle": { },
		...
		".zshenv": { }
	},
	"root-gitignore": [
		".DS_Store",
		"dotfiles/.config/nvim/.netrwhist",
		"dotfiles/.config/nvim/spell/en.utf-8.add",
		"dotfiles/.config/ranger/plugins/ranger_devicons",
		"dotfiles/.config/zsh/.zcompdump*",
		"dotfiles/.pypirc",
		"dotfiles/.ssh"
	],
	"dotfiles-gitignore": [
		".DS_Store",
		".config/nvim/.netrwhist",
		".config/nvim/spell/en.utf-8.add*",
		".config/ranger/plugins/*",
		".config/zsh/.zcompdump*",
		".config/zsh/.zinit",
		".config/tmux/plugins",
		".config/tmux/resurrect",
		".pypirc",
		".ssh/*"
	],
	"config_mapping": {
		"/Users/alichtman/Library/Application Support/Sublime Text 2": "sublime2",
		"/Users/alichtman/Library/Application Support/Sublime Text 3": "sublime3",
		"/Users/alichtman/Library/Application Support/Code/User/settings.json": "vscode/settings",
		"/Users/alichtman/Library/Application Support/Code/User/Snippets": "vscode/Snippets",
		"/Users/alichtman/Library/Application Support/Code/User/keybindings.json": "vscode/keybindings",
		"/Users/alichtman/Library/Preferences/com.apple.Terminal.plist": "terminal_plist"
	}
}
```

### Git Integration

---

#### A Word of Caution

This backup tool is git-integrated, meaning that you can easily store your backups remotely (on GitHub, for example.) Dotfiles and configuration files may contain sensitive information like API keys and ssh keys, and you don't want to make those public. To make sure no sensitive files are uploaded accidentally, `shallow-backup` creates a `.gitignore` file if it can't find one in the directory. It excludes `.ssh/` and `.pypirc` by default. It's safe to remove these restrictions if you're pushing to a remote private repository, or you're only backing up locally. To do this, you should clear the `.gitignore` file without deleting it.

_If you choose to back up to a public repository, look at every file you're backing up to make sure you want it to be public._

> [!NOTE]
> As of `v6.2`, `trufflehog` is run as a required precommit hook and will detect secrets.

#### .gitignore

As of `v4.0`, any `.gitignore` changes should be made in the `shallow-backup` config file. `.gitignore` changes that are meant to apply to all directories should be under the `root-gitignore` key. Dotfile specific gitignores should be placed under the `dotfiles-gitignore` key. The original `default-gitignore` key in the config is still supported for backwards compatibility, however, converting to the new config format is strongly encouraged.

### Output Structure

---

```shell
shallow_backup/
├── configs
│   ├── plist
│   │   └── com.apple.Terminal.plist
│   ├── sublime_2
│   │   └── ...
│   └── sublime_3
│       └── ...
├── dotfiles
│   ├── .bash_profile
│   ├── .bashrc
│   ├── .gitconfig
│   ├── .pypirc
│   ├── ...
│   ├── shallow-backup.json
│   ├── .ssh/
│   │   └── known_hosts
│   ├── .vim/
│   └── .zshrc
├── fonts
│   ├── AllerDisplay.ttf
│   ├── Aller_Bd.ttf
│   ├── ...
│   ├── Ubuntu Mono derivative Powerline Italic.ttf
│   └── Ubuntu Mono derivative Powerline.ttf
└── packages
    ├── brew-cask_list.txt
    ├── brew_list.txt
    ├── cargo_list.txt
    ├── gem_list.txt
    ├── installed_apps_list.txt
    ├── npm_list.txt
    ├── macports_list.txt
    ├── pip_list.txt
    └── sublime3_list.txt
```

### Reinstalling Dotfiles

---

To reinstall your dotfiles, clone your dotfiles repo and make sure your shallow-backup config path can be found at either `~/.config/shallow-backup.json` or `$XDG_CONFIG_HOME/.shallow_backup.json`. Set the `backup-path` key in the config to the path of your cloned dotfiles. Then run `$ shallow-backup -reinstall-dots`.

When reinstalling your dotfiles, the top level `.git/`, `.gitignore`, `img/` and `README.md` files and directories are ignored.

### Want to Contribute?

---

Check out `CONTRIBUTING.md` and the `docs` directory.
