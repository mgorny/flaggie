flaggie -- an another flag mangler
==================================


Quick introduction
------------------

flaggie is just another handy, CLI-based mangler for `package.*` files.
Although it is originally based on [flagedit][1] by Damien Krotkine, it
aims at being nicer to use and more powerful than flagedit.

Key features:
- use of portage API (you no longer have to specify the category even
	the first time you're adding a flag!),
- support for `package.*` directories (most actions performed by flaggie
	affect the effective declarations, in whichever of the files they
	appear),
- support for `package.license` file,
- smart recognition of action arguments -- flaggie knows whether you're
	passing a USE flag or a keyword,
- extensive support for `make.conf` file syntax (incl. `USE_EXPAND`).

[1]: http://damien.krotkine.com/the-player-of-games/flagedit.html


Synopsis
--------

Symbolically, the flaggie synopsis could be written as:

	flaggie [<options>] [<global-actions>] [<packages> <actions>] [...]

In other words, the basic arguments to `flaggie` consist of package
specifications and action specifications grouped together. Each group
of actions is applied to the package group preceding it; and if a group
of actions precedes the first package group, it is considered a global
action group and these actions are supposedly performed on the variables
in the `make.conf` file.

A package specification has the same atom format as one used by emerge.
You can provide a simple package name as well as a complete,
version-restricted atom. If you don't specify the package category,
flaggie will try to grab it using Portage API.

An action specification consists of an action symbol and an optional or
obligatory argument (flag). The action symbols are:

- `+` to enable a flag,
- `-` to disable a flag,
- `%` to reset the flag(s) to the default state (remove them completely
	from the files),
- `?` to print the effective flag declaration from the files.

Each of the actions should be followed by an argument. It could be
a flag name, a keyword or a license string. Arguments can be shell
patterns (same as in filename matching). If no argument is passed,
flaggie assumes `?*`.

With `+` and `-` actions, pattern matching is performed against package
`IUSE`, `KEYWORDS` or `LICENSE` variable. With `%` and `?`, it is done
against values specified in `package.*` files.

Please denote that for keywords, `*` and `**` arguments have special
meaning and will not be parsed as patterns. If you need to perform
pattern matching there, please use `?*` instead.

In addition to that, the argument can be preceded by a namespace
specifier in the form `ns::`, where `ns` can be one of `use`, `kw` or
`lic` (or a pattern).

When no namespace is specified, the namespace is guessed from the actual
argument if it is not a pattern; `use` is assumed otherwise.


Cleanup actions
---------------

Except for the specific actions, flaggie can be perform a set of cleanup
actions which are done on whole `package.*` files. These are enabled
using long options.

The following cleanup actions are supported:

- `--drop-ineffective` to drop all flag declarations which are
	considered ineffective. In other words, those which are overriden
	by the entries or flags following them.

	In other words, the following example file:

		app-misc/foo bar baz
		app-misc/foo -bar bar

	would be written as:

		app-misc/foo baz
		app-misc/foo bar

- `--sort-entries` to sort all entries in the file by package name.

- `--sort-flags` to sort all flags (keywords, licenses) in the entries
	by their basename.

- `--drop-unmatched-pkgs` to remove all `package.*` file entries
	referring to packages not having a match in portdb (thus, either
	being a typo, outdated or coming from a removed repo).

- `--drop-unmatched-flags` to remove all flags, keywords and licenses
	which do not match the package metadata (`IUSE`, `KEYWORDS`
	and `LICENSE` keys respectively). In other words, this should remove
	outdated flags. Please note that, in order to avoid mistakes, this
	action won't remove flags for packages which do not have a match
	in portdb (`--drop-unmatched-pkgs` is useful for that).

In addition to the actual cleanup actions, a set of shorthand options is
available too:

- `--sort` to sort both the entries itself and their flags,
- `--cleanup` to perform sorting and drop ineffective (redundant) flag
	declarations,
- `--destructive-cleanup` to perform all of the cleanup actions
	listed above.

There is also a single special option:

- `--migrate-files` to upgrade all the outdated files to the latest
	formats used by Portage. Right now, this involves dropping
	`package.keywords` and moving all its entries
	to `package.accept_keywords`.


Examples
--------

1. Enabling `USE=doc` for `x11-libs/gtk+:2`:

		flaggie gtk+:2 +doc

2. Keyword-unmasking `sys-apps/portage-2.2` (omitting the live version):

		flaggie '<portage-9999' '+**'

3. Resetting all USEflags of `net-im/ekg2` to their default values:

		flaggie ekg2 %

4. Easy license-unmasking `www-plugins/adobe-flash`:

		flaggie adobe-flash +lic::

5. Performing a cleanup of `package.*` files:

		flaggie --cleanup

6. Enabling all devices for `app-misc/lirc` (e.g. for testing):

		flaggie app-misc/lirc '+lirc_devices_*'

<!-- vim:se syn=markdown : -->
