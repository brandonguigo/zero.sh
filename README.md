[![CircleCI](https://circleci.com/gh/brandonguigo/usanidi.svg?style=svg)][CI]
[![Latest Release](https://img.shields.io/github/v/release/usanidi/usanidi?sort=semver&style=flat-square)][Release]

[CI]: https://travis-ci.org/usanidi/usanidi
[Release]: https://github.com/usanidi/usanidi/releases

## Usanidi - Configure your laptop with Code

## Overview

This program lets you configure your laptop (only for Mac, for now...) with code.
You are able to define workspaces that you can install as you wish.

For now, you can only install workspaces, but not remove them, nor replace symlinked directories.

## Usage

Instead, just run the following:

```sh
# Clone your configuration repo to your preferred location. For example:
git clone --recursive <YOUR_REPO> ~/.usanidi

# If on a new machine, once finished, restart and run again to ensure system 
# settings are applied correctly.
nidi setup
```

... and you'll be back up and running, with all of your applications and
command line utilities re-installed (and configurations restored).

**Note**: By default, `nidi` assumes your configuration directory is located in
one of following locations. This can be changed with the `--directory` flag.

1. `$XDG_CONFIG_HOME/usanidi/config`
2. `$HOME/.config/usanidi/config`
3. `$HOME/.usanidi/config`

## 

During setup, you may be asked for your password as some commands require admin
privileges. Each of these will be printed before running.

The `setup` command will do the following, in order:

1. Check for system and application updates via
   [`softwareupdate`][softwareupdate], [`brew`][brew], [`brew cask`][brew
   cask], and [`mas`][mas].
2. Install packages and applications via [Homebrew Bundle].
3. Run any scripts under `run/before` in alphabetical order.
4. Apply system defaults described in `defaults.yaml` via
   [`apply-user-defaults`][apply-user-defaults].
5. Symlink configuration files listed under `symlinks` to the home directory.
6. Run the remaining scripts under `run/after` in alphabetical order.

[softwareupdate]: https://tldr.ostera.io/osx/softwareupdate
[brew]: https://brew.sh
[brew cask]: https://github.com/Homebrew/homebrew-cask
[mas]: https://github.com/mas-cli/mas
[Homebrew Bundle]: https://github.com/Homebrew/homebrew-bundle
[apply-user-defaults]: https://github.com/usanidi/apply-user-defaults

This command is idempotent, and can be safely invoked again to update tools and
ensure everything has been installed correctly.

It will **not** wipe over files that already exist when symlinking or at any
other point in the process, aside from what is done by system upgrade tools or
in your own custom before & after scripts.

In addition, there is **no magic** done by this tool. Each command is printed
before it is run.

## Directory Structure

The directory structure in `~/.usanidi` (or wherever you choose to store it)
is expected to look like this:

```
- Brewfile # Homebrew Bundle dependency file.
- defaults.yaml # macOS defaults to be set by apply-user-defaults command.
- symlinks/
    -> name/ # Arbitrary alias, for example "zsh", "vim", etc.
        => file or directory # Exact name of file or directory to symlink.
- run/
    -> before/
        => [ ... executable scripts ... ]
    -> after/
        => [ ... executable scripts ... ]
```

## Workspaces

Multiple machine configurations can be managed via the following directory
structure:

```
- workspaces/
  -> shared/
  -> home/
  -> work/
```

This will first apply the setup described in `shared`, followed by `home` or
`work` when specifying a workspace argument via:

```
~/.usanidi/nidi/setup [home|work]
```

It can also recurse, for example:

```
- workspaces/
  -> shared/
  -> home/
    => workspaces/
      -> shared/
      -> desktop/
      -> laptop/
  -> work/
```

This describes three workspaces, `home.desktop`, and `home.laptop`, and `work`.

It will run the same series of steps as before, but first setup the workspace
described in `shared` of the parent or sister directory. For example, when
running `nidi/setup home.desktop`, it will do the following:

1. Check for system and application updates.
2. For each workspace in `workspaces/shared`, followed by in
      `workspaces/home/workspaces/shared`, then
      `workspaces/home/workspaces/desktop`:
   - Install packages and applications via Homebrew Bundle.
   - Run any scripts under `run/before` in alphabetical order.
   - Apply system defaults described in `defaults.yaml` via apply-user-defaults.

... etc., for each of the steps listed above.

## Additional Features

Usanidi also supports running any of the above steps independently.

```
Usage: nidi <command> [options]

Radically simple personal bootstrapping tool for macOS.

Commands:
  setup             Setup a workspace, or directory if none is given
  update            Check for system and application updates
  bundle            Run brew bundle on a workspace
  apply-defaults    Apply user defaults for a workplace
  apply-symlinks    Apply symlinks for a workplace
  run-scripts       Run before & after scripts for a workplace
  help              Prints help information
  version           Prints the current version of this app
```

## Installation

### Homebrew

[Homebrew](https://brew.sh) is the preferred way to install:

```sh
$ brew install usanidi/tap/zero
```

Alternatively, pre-compiled binaries are available on the [releases
page](https://github.com/usanidi/usanidi/releases).

### Building from Source

To build from source, run:

```
$ git clone https://github.com/brandonguigo/usanidi.git usanidi
$ cd usanidi
$ make archive
# make install
```

To develop locally, instead use `swift run usanidi`.

### Submodule

Since `zero` requires Homebrew for installation, it needs a [helper
script](setup) for the initial setup. This repo contains one that can either be
copied or included as a submodule:

```sh
cd ~/.dotfiles
git submodule add https://github.com/usanidi/usanidi zero
```

Then, to pin to the latest stable version, run:

```sh
git submodule update --init --remote zero
git -C zero checkout <VERSION>
git commit
```

Now when bootstrapping your new machine, you can simply run:

```sh
git clone --recursive <YOUR_REPO> ~/.dotfiles
~/.dotfiles/usanidi/setup
```

This gives you a single script that can be used to fully setup your new
machine.

It is equivalent to running the `usanidi setup` command (and will accept any
respective arguments, e.g. workspaces), only it first installs Homebrew and
`usanidi`. Just like `usanidi setup`, it is idempotent and can be safely run again.

Note that it may be necessary run `git submodule update --init` later when
pulling in the new submodule into an existing repo, unless the `--recursive`
flag is included when cloning as shown above.

## Working examples

To see how this works out in practice, here are some repos that use `usanidi`:

- [msanders/setup](https://github.com/msanders/setup)

> Add yours here — send a PR.

## Roadmap & missing features

- Linux support. This should be pretty straightforward, but requires accounting
  for additional system update tools and package managers that I haven't had
  time for yet. It will also be limited to distributions that support Swift.

- Currently it's not possible to specify a target for symlinks; they are just
  all expanded to the home directory, matching the nested directory structure
  they are contained in under `symlinks/`. This works fine for my use-case but
  not sure if it will be enough for others.

- GNU Stow is a neat tool, but doesn't offer the same level of utility or error
  handling that Cider previously did. It would be nice to offer a more modern
  alternative.

**Note**: `usanidi` is a work-in-progress, but it's fairly well-tested and
should be kind to your machine.

## Dependencies

The following dependencies are required & installed when building the brew
formula:

- [`apply-user-defaults`](https://github.com/usanidi/apply-user-defaults)
  installed via Homebrew.
- [`mas`](https://github.com/mas-cli/mas) installed via Homebrew.
- [`stow`](https://www.gnu.org/software/stow/) installed via Homebrew.

In addition, [Homebrew](https://brew.sh) and Xcode Command Line Tools are
required.

## Non-Goals

This tool is intended to be a very minimal approach to personal system
configuration. If you are looking for something more full-featured, e.g. that
provides a complex CLI or additional features for managing many machines at
once, there are other solutions available. In my experience just dealing with
my own machines, this was all that was necessary.

## Contributing

If you are interested in this project, please consider contributing. Here are a
few ways you can help:

- [Report issues](https://github.com/usanidi/usanidi/issues).
- Fix bugs and [submit pull requests](https://github.com/usanidi/usanidi/pulls).
- Write, clarify, or fix documentation.
- Suggest or add new features.
- Star this repository so it can be added to upstream Homebrew.

### Setup

```bash
git clone https://github.com/brandonguigo/usanidi.git usanidi
yarn #to install the prehooks for commit messages and pre-commit lint
```

## Inspiration

This is partly inspired by [@gerhard's setup][1], in addition to [this blog
post][2] on GNU Stow by Brandon Invergo. It is structurally based on (and the
spiritual successor to) [Cider][3].

[1]: https://github.com/gerhard/setup
[2]: http://brandon.invergo.net/news/2012-05-26-using-gnu-stow-to-manage-your-dotfiles.html
[3]: https://github.com/msanders/cider

## License

usanidi is licensed under the MIT License. See [LICENSE](LICENSE) for more
information.
