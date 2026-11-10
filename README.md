# tint

One terminal color scheme, applied to the whole desktop.

A scheme is 16 ANSI colors plus a background and foreground: any of the 365
[Gogh](https://github.com/Gogh-Co/Gogh) schemes in `data/gogh.json`, or a
kitty-format file in `~/.config/tint/schemes/`. `tint apply NAME` renders every
template listed in `~/.config/tint/targets.toml` with that palette and reloads
what can reload live.

```sh
tint list tokyo          # find a scheme
tint apply "Tokyo Night" # everything
tint apply Vague --live  # only the instant targets (a picker's preview)
tint apply Vague --root /tmp/x   # render elsewhere, touch nothing
tint check               # every template renders, every block has its markers
```

## How it is laid out

- `~/.config/tint/targets.toml`: one entry per file tint writes.
- `~/.config/tint/templates/`: the files themselves, with `{{key}}` where a
  color goes. `mode = "file"` targets are owned whole; `mode = "block"` targets
  own only the lines between `tint:begin` and `tint:end`.
- `~/.config/tint/schemes/*.conf`: your own schemes. Extra keys (for example
  `syntax_function`) override what tint would derive.
- `~/.config/tint/state.json`: the applied scheme and the black-background flag.

Keys: `background`, `foreground`, `color0`-`color15`; neutrals stepped from
background toward foreground (`surface`, `sunken`, `overlay`, `hairline`,
`selection`, `subtle`); roles (`accent`, `ok`, `warn`, `crit`, `info`, `agent`,
`muted`); `syntax_*`; `name`. Filters: `bare`, `upper`, `rgb`, `led`,
`alpha:NN`.

`--black` (the default, remembered) forces a `#000000` background.

## To add a target

Copy the config to `templates/`, replace its colors with keys, add a
`[[target]]`, run `tint check`, then `tint apply` the current scheme with
`--root` and diff the result against the real file.
