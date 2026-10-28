# hypr-sens

Save and apply Hyprland's global mouse sensitivity with the `sens` command. With optional libratbag support, `sens` also shows and changes hardware DPI(dots per inch).

## Install manually

Requires Linux, Z shell, GNU coreutils, and a running Hyprland session for queries, first-time initialization, and live changes. Lua mode requires a Hyprland version that supports `hyprctl eval` and `hl.config`. Legacy mode requires support for `hyprctl keyword input:sensitivity`.

On Arch Linux, install the dependencies and Git for the clone:

```zsh
sudo pacman -S --needed hyprland zsh coreutils git
git clone https://github.com/OwaisQuadri/hypr-sens.git
cd hypr-sens
sudo install -Dm755 sens /usr/local/bin/sens
sudo install -Dm644 LICENSE /usr/local/share/licenses/hypr-sens/LICENSE
sudo install -Dm644 README.md /usr/local/share/doc/hypr-sens/README.md
```

The package name is `hypr-sens`. The executable is `sens`.

For optional hardware DPI support on Arch Linux:

```zsh
sudo pacman -S --needed libratbag
```

Hardware access requires libratbag's `ratbagctl`, the `ratbagd` daemon, and a supported mouse. Arch's package provides D-Bus activation for `ratbagd`; the daemon must be available to your session. Hyprland sensitivity commands do not require libratbag or a supported mouse.

### Arch User Repository package

The `hypr-sens` AUR(Arch User Repository) package is not yet published. Use the manual installation above for now.

The [PKGBUILD](packaging/PKGBUILD) downloads `sens`, `LICENSE`, and `README.md` from the pinned GitHub tag `v0.1.0`. Each file has a SHA-256(Secure Hash Algorithm, 256-bit) checksum. The package metadata resides in `packaging/.SRCINFO`.

The package installs `/usr/bin/sens`, `/usr/share/licenses/hypr-sens/LICENSE`, and `/usr/share/doc/hypr-sens/README.md`. Installation does not initialize sensitivity or edit any user configuration.

## Set up persistence

Choose the format used by your root Hyprland config. Do not initialize both formats.

### Lua config

```zsh
sens init --lua
```

The command creates `${XDG_CONFIG_HOME:-$HOME/.config}/hypr/sens.lua` from the running global sensitivity. It prints a `dofile("...")` line with an escaped absolute path.

1. Add the printed line once at the **end** of your root Lua config, after other settings and includes.
2. Reload Hyprland:

```zsh
hyprctl reload
```

### Legacy config

```zsh
sens init --legacy
```

The command creates `${XDG_CONFIG_HOME:-$HOME/.config}/hypr/sens.conf` from the running global sensitivity. It prints a `source = ...` line with an absolute path.

1. Add the printed line once at the **end** of your root legacy config, after other settings and includes.
2. Reload Hyprland:

```zsh
hyprctl reload
```

Initialization never edits your root config or changes the running sensitivity. Repeating initialization for the same format keeps the saved value and prints the include instruction again, even without a running session. Persistence requires you to add that include and reload. A later global sensitivity setting in your config can override the snippet.

## Use sens

```zsh
sens
sens 0.5
sens -0.25
sens 0
sens --dpi 1600
sens --help
sens --version
```

With no arguments, `sens` shows the running global `input:sensitivity` through native `hyprctl getoption` output and the current hardware DPI through `ratbagctl`. The Hyprland value is not a query of the saved snippet or of an individual device. If DPI support is unavailable, `sens` prints a brief reason without failing the sensitivity query.

A number saves and applies a global sensitivity between `-1` and `1`, inclusive. Decimal notation such as `0`, `-0.25`, and `.5` is accepted. A leading `+`, exponent notation, expressions, and values outside that range are rejected.

Each change atomically replaces the one managed snippet. Lua mode applies the value with `hyprctl eval 'hl.config(...)'`. Legacy mode uses `hyprctl keyword input:sensitivity VALUE`. `sens` never appends overrides or edits another config file.

Explicit per-device Hyprland sensitivities take precedence over the global setting. `sens` does not change those settings or acceleration profiles.

## Change hardware DPI

`sens --dpi VALUE` accepts a positive decimal integer. It discovers mice through `ratbagctl list` and checks DPI capability with `dpi get-all`. Exactly one mouse must report configurable DPI. No device is selected when several supported mice are connected.

The command uses `ratbagctl ID dpi set VALUE`, which commits by default to the current active profile and resolution. It does not use `--nocommit`, write a DPI config file, or change profiles, polling rates, buttons, or Hyprland sensitivity. Persistence uses the mouse and driver's commit support, not the Hyprland snippet. A later profile or resolution switch can select a different DPI.

The driver validates the requested DPI. `dpi get-all` is not treated as an exhaustive whitelist because some drivers accept values between the listed steps. After setting DPI, `sens` reads it back and requires the exact output `VALUEdpi` to report success. Mice that return a different format are displayed as reported by bare `sens`, but changes cannot be verified through this command.

Missing tools, daemon access failures, unsupported mice or requested values, multiple supported mice, and failed read-back checks produce clear errors. A failed or mismatched read-back does not undo a hardware change. Read the mouse's current state before retrying.

## Paths and failures

- The config root is `${XDG_CONFIG_HOME:-$HOME/.config}`. It must be absolute and cannot contain a newline or carriage return. Ordinary spaces are supported. Keep `XDG_CONFIG_HOME` consistent between initialization and later invocations.
- Lua include paths escape backslashes, double quotes, and tabs. Legacy paths cannot contain `#`, `$`, `*`, `?`, `[`, `]`, or a backslash because those characters can alter how the include is parsed.
- `sens` does not discover your root config or follow Hyprland's `--config` option. You must add the printed include to the root config your session actually uses.
- If both `sens.lua` and `sens.conf` exist, initialization and sensitivity changes fail rather than select a format. Keep only the snippet your root config includes. To switch formats, remove the old include and snippet, then initialize the other format.
- Managed snippets must be regular files, not symlinks. Treat their contents as generated. A sensitivity change replaces the entire snippet.
- If saving fails, `sens` does not attempt a live change. If saving succeeds but the live change fails, `sens` exits unsuccessfully and explicitly reports that the value was saved but not applied. The snippet remains saved for a later reload through the include.
- Writes are atomic, but concurrent `sens` invocations are not serialized. Run one change at a time.
- Invalid arguments exit with status `2`. Setup, persistence, DPI, and live-apply failures exit with status `1`. A no-argument query returns `hyprctl`'s exit status regardless of optional DPI availability. `--dpi` does not require a Hyprland session or an initialized snippet.

## License

[MIT](LICENSE).
