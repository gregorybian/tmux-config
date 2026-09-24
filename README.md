# dotfiles

Personal configuration files, tracked in git and symlinked into place.

The real file lives here; the path the program expects is a symlink pointing at
it. That way the config is version-controlled without the repository having to
sit in `$HOME`.

## Contents

| File | Symlinked to | Purpose |
|---|---|---|
| `tmux.conf` | `~/.tmux.conf` | tmux: mouse, scrollback, panes, status bar |

## Install on a new machine

```sh
git clone <this-repo> ~/dotfiles
ln -s ~/dotfiles/tmux.conf ~/.tmux.conf
```

If `~/.tmux.conf` already exists, move it aside first — `ln -s` will not
overwrite an existing file.

## Applying tmux changes

tmux reads its config only at server start, so edits need an explicit reload:

```sh
tmux source-file ~/.tmux.conf
```

`prefix` + `r` is bound to the same thing.

Note that `source-file` only adds and overrides bindings. It does not remove a
binding deleted from the file — the running server keeps it until restart. Add
an explicit `unbind` when removing one.

## tmux notes

- `mode-keys` selects which key table is live: `copy-mode` (emacs, the default)
  or `copy-mode-vi`. A binding written to the inactive table silently does
  nothing, so mouse overrides are bound in both.
- `MouseDragEnd1Pane` defaults to `copy-pipe-and-cancel`. The `cancel` exits
  copy-mode on every drag release, which reads as the pane snapping to the
  bottom. Overridden with `copy-selection-no-clear`.
- `WheelUpPane` defaults to handing the wheel to any application that enabled
  mouse reporting, so full-screen programs never reach scrollback. Overridden
  to always enter copy-mode.
