# tint

Apply one terminal color scheme to every app on a Linux desktop.

A scheme is 16 ANSI colors plus a background and a foreground. `tint` renders
that palette into every config file you list, from templates you write once,
and reloads whatever can reload live. Terminal, editor, compositor, toolkits,
prompt, keyboard LEDs: one command changes them together.

## Features

- **365 schemes included.** The [Gogh](https://github.com/Gogh-Co/Gogh)
  collection ships in `data/gogh.json`. Your own schemes are kitty-format
  files in `~/.config/tint/schemes/`.
- **Templates, not themes.** A target is one of your real config files with
  `{{key}}` where a color goes. `tint` owns the whole file, or only the lines
  between two markers in a file you maintain by hand.
- **Roles and neutrals are derived.** Beyond the raw colors, a template can
  ask for an accent, warning and critical colors, and surface, hairline and
  selection tones stepped from the background toward the foreground. Any
  scheme works, dark or light.
- **Live preview.** Targets marked `live` are cheap enough to apply while
  you browse; `--live` writes only those.
- **A look on top of any scheme.** Corner radius and border thickness are
  template keys, set once for everything that draws a box, and the accent
  and secondary can be pinned to any of the scheme's six hues.
- **Safe to try.** `--root DIR` renders the whole set somewhere else and
  touches nothing; `tint check` proves every template still renders.

## Install

### Requirements

- Python 3.11 or newer (it uses `tomllib`). No other dependencies.

```sh
git clone https://github.com/slastra/tint ~/Projects/tint
ln -s ~/Projects/tint/tint ~/.local/bin/tint
mkdir -p ~/.config/tint
cp -r ~/Projects/tint/examples/* ~/.config/tint/
```

The example config has one target, kitty. Add `include colors.conf` to
`kitty.conf` to use it.

## Usage

```sh
tint list tokyo                 # find a scheme by name
tint list --light               # only light schemes (--dark for the others)
tint apply "Tokyo Night"        # every target, then reload
tint apply Vague --live         # only the live targets (a picker's preview)
tint apply Vague --root /tmp/x  # render under /tmp/x, touch nothing
tint apply Vague --black        # force a #000000 background (remembered)
tint look --radius 4 --border 1   # corner radius and border thickness
tint look --accent blue --secondary auto   # which of the scheme's hues play which part
tint current                    # what is applied now
tint show Vague                 # the resolved palette as JSON
tint check                      # every template renders, every block has markers
```

`tint list --json` prints every scheme with a few swatch colors, whether it
is dark, and the current one. It exists for pickers.

## Configuration

Everything lives in `~/.config/tint/`.

| Path | What it is |
|---|---|
| `targets.toml` | One `[[target]]` per file `tint` writes. |
| `templates/` | The files themselves, with `{{key}}` where a value goes. |
| `schemes/*.conf` | Your own schemes, in kitty's format. |
| `state.json` | The applied scheme, the black flag, and the look. |

A target:

```toml
[[target]]
template = "kitty-colors.conf"       # in templates/
dest = "~/.config/kitty/colors.conf"
reload = "pkill -USR1 -x kitty"      # optional, runs after a change
live = true                          # optional, include in --live
mode = "file"                        # or "block"
```

`mode = "block"` replaces only the lines between `tint:begin` and `tint:end`
in `dest` (set `begin` and `end` to use other markers), so `tint` can own a
few lines of a file you otherwise edit by hand. Reload commands run once
each, after every target is written.

Template keys:

| Keys | |
|---|---|
| `background`, `foreground`, `color0` to `color15` | The scheme as given. |
| `surface`, `sunken`, `overlay`, `hairline`, `selection`, `subtle` | Neutrals: the background stepped toward the foreground. |
| `accent`, `ok`, `warn`, `crit`, `info`, `agent`, `muted` | Roles. On a dark background they take the bright ANSI variants. |
| `syntax_comment`, `syntax_keyword`, and the other `syntax_*` | Editor colors, for schemes that say nothing finer. |
| `name`, `appearance` | The scheme's name, and `dark` or `light`. |
| `secondary` | A second color for small print, kept readable against the surface. |
| `radius`, `border` | Whole pixels, from `tint look`. |

A scheme file may set any derived key itself, and that value wins.

Filters follow a key after `|`: `bare` (no `#`), `upper`, `rgb`
(`r, g, b`), `led` (deeper saturation, for LEDs), `alpha:NN` (`#rrggbbNN`).
For example `{{accent|bare}}` or `{{background|alpha:cc}}`.

To add a target: copy the config into `templates/`, replace its colors with
keys, add a `[[target]]`, run `tint check`, then apply the current scheme
with `--root` and diff the result against the real file.

## How it works

`tint` is one Python script. `apply` resolves the scheme, derives the
palette, renders each template, and writes a destination only when its
contents changed, through a temporary file and a rename so a watcher never
sees half a file. Only the targets that changed have their reload command
run.

A working configuration with about twenty targets (Hyprland, GTK, Qt,
Neovim, Zed, fish, Starship, mpv, a keyboard, an icon and cursor theme) is
in [dots](https://github.com/slastra/dots). The scheme picker in
[lastshell](https://github.com/slastra/lastshell) is built on `tint list
--json` and `tint apply --live`.

## Credits

- The schemes in `data/gogh.json` are from
  [Gogh](https://github.com/Gogh-Co/Gogh) (MIT), each credited to its author
  in the file.

## License

MIT © 2026 Shaun Lastra. See [LICENSE](LICENSE).
