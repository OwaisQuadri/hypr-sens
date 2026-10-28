# hypr-sens

Save Hyprland mouse sensitivity from your terminal. The `sens` command applies each change now and keeps it across reloads through one configuration snippet.

```sh
sens
sens -0.5
sens --dpi 1600
```

DPI(dots per inch) controls the mouse's hardware resolution. Desktop sensitivity controls Hyprland's pointer speed. Neither setting replaces a game's raw-input sensitivity.

## Install on Arch Linux

Install the package directly from [release v0.1.0](https://github.com/OwaisQuadri/hypr-sens/releases/tag/v0.1.0):

```sh
sudo pacman -U https://github.com/OwaisQuadri/hypr-sens/releases/download/v0.1.0/hypr-sens-0.1.0-1-any.pkg.tar.zst
```

The package name is `hypr-sens`. The installed command is `sens`.

This is a direct package download, not an official Arch repository package. The package is not available through AUR(Arch User Repository). `pacman -S hypr-sens` does not work without a configured package repository.

The package installs the command, license, and documentation. It does not edit your Hyprland configuration.

## Set up persistence

Run setup inside your Hyprland session, without `sudo`.

1. Choose the command that matches your root Hyprland configuration.

   For Lua configuration:

   ```sh
   sens init --lua
   ```

   For legacy configuration:

   ```sh
   sens init --legacy
   ```

2. Add the printed include line once at the end of your root Hyprland configuration, after other settings and includes.
3. Reload Hyprland:

   ```sh
   hyprctl reload
   ```

Initialization copies the current global sensitivity into `sens.lua` or `sens.conf` under `${XDG_CONFIG_HOME:-$HOME/.config}/hypr`. It prints the matching include line. It never edits your root configuration or overwrites an existing saved value.

Choose one format, not both. Repeating initialization for that format keeps your saved value. Persistence requires the include line.

## Adjust desktop sensitivity

Check your current settings:

```sh
sens
```

With a supported mouse and libratbag available, the output includes both values. For example:

```text
Global Hyprland sensitivity:
float: -0.800000
set: true
Hardware DPI: 6400dpi
```

Set a global sensitivity between `-1` and `1`:

```sh
sens -0.5
```

Negative values slow the pointer. Positive values speed it up. `sens 0` selects Hyprland's neutral sensitivity setting.

Each change replaces the managed snippet and applies the value to the running session. The command accepts decimal numbers such as `-.5` and `0.25`. It rejects a leading `+`, exponent notation, and values outside the range.

Explicit per-device sensitivity settings override the global value. Remove a device-specific sensitivity override from your own configuration if you want that device to follow `sens`. The command does not change per-device settings or acceleration profiles.

Minecraft and other games with raw input can bypass desktop pointer sensitivity. Adjust the game's sensitivity or its input tool separately.

## Change hardware DPI

On Arch Linux, install the optional dependency:

```sh
sudo pacman -S --needed libratbag
```

Hardware access requires `ratbagctl`, the `ratbagd` daemon, and a supported mouse. Arch's package provides activation through D-Bus(Desktop Bus).

Set a positive integer DPI value supported by your mouse:

```sh
sens --dpi 1600
```

The command commits the value to the mouse's active profile and resolution, then checks the reported value. It does not change Hyprland sensitivity, profiles, polling rates, or buttons. Hardware persistence depends on the mouse and driver. Switching profiles or resolutions can select a different DPI.

Connect only one mouse with configurable DPI when you use this command. If several supported mice are connected, `sens` refuses to choose one.

Without hardware support, bare `sens` still queries desktop sensitivity and explains why DPI is unavailable. `sens --dpi` requires hardware support but does not require a Hyprland session or a managed snippet.

## Install from source

The command requires Linux, Z shell, GNU(GNU's Not Unix) coreutils, and Hyprland. Lua mode requires `hyprctl eval` and `hl.config`. Legacy mode requires `hyprctl keyword input:sensitivity`.

On Arch Linux:

```sh
sudo pacman -S --needed hyprland zsh coreutils git
git clone https://github.com/OwaisQuadri/hypr-sens.git
cd hypr-sens
sudo install -Dm755 sens /usr/local/bin/sens
sudo install -Dm644 LICENSE /usr/local/share/licenses/hypr-sens/LICENSE
```

Then follow [Set up persistence](#set-up-persistence).

For Arch package builds, use [PKGBUILD](packaging/PKGBUILD) and its generated [package metadata](packaging/.SRCINFO). The recipe pins the released source files to `v0.1.0` with checksums. Changes to the README on the main branch do not change that release.

## Resolve setup problems

### Changes disappear after a reload

Check that the printed include line appears at the end of the root configuration your session actually uses. A later global sensitivity setting can override the snippet.

`sens` does not discover your root configuration or follow Hyprland's `--config` option. Keep `XDG_CONFIG_HOME` consistent between initialization and later commands.

### Both configuration formats exist

Keep the format your root configuration includes. To switch formats, remove the old include and its managed snippet. Initialize the other format, then add its printed include line.

Managed snippets must be regular files, not symbolic links. Treat their contents as generated because each sensitivity change replaces the entire file.

### The configuration path is rejected

Use an absolute configuration root without newline or carriage-return characters. Spaces are supported. Legacy paths cannot contain `#`, `$`, `*`, `?`, `[`, `]`, or a backslash because those characters can change how Hyprland reads the include.

### A change reports an error

If saving fails, `sens` does not attempt a live change. If saving succeeds but live application fails, the error states that the value remains saved. Restore access to your Hyprland session before reloading the configuration.

A failed hardware read-back can follow an actual DPI change. Check the mouse's current state before you retry. The current verifier requires the driver to report the exact format `VALUEdpi`.

Run one change at a time. File replacement is atomic, but concurrent commands are not serialized.

For command syntax, run `sens --help`. Invalid arguments return exit status `2`. Setup, save, hardware-change, and live-apply failures return `1`. Bare `sens` returns the desktop query's exit status regardless of optional DPI availability.

## License

[MIT(Massachusetts Institute of Technology) license](LICENSE).
