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

### 1. Requirements

- **tmux 3.2 or newer.** `extended-keys` and `terminal-features` do not exist
  in earlier versions and the config will error on load. Check with `tmux -V`.
- **A clipboard helper.** The copy binding pipes to `xclip`; without it the
  binding runs and silently copies nothing.

```sh
sudo apt-get install -y xclip          # X11
sudo apt-get install -y wl-clipboard   # Wayland — see note below
```

Check which display server you are on with `echo $XDG_SESSION_TYPE`. On
Wayland, `xclip` only works if XWayland is present; otherwise replace every
`xclip -in -selection clipboard` in `tmux.conf` with `wl-copy`.

### 2. Link the config

```sh
git clone <this-repo> ~/dotfiles
ln -s ~/dotfiles/tmux.conf ~/.tmux.conf
```

If `~/.tmux.conf` already exists, move it aside first — `ln -s` will not
overwrite an existing file.

### 3. Free up Ctrl+Shift+V (GNOME Terminal only)

GNOME Terminal claims `Ctrl+Shift+V` for paste and never forwards it to tmux,
so the copy binding cannot fire until the shortcut is disabled:

```sh
gsettings set \
  "org.gnome.Terminal.Legacy.Keybindings:/org/gnome/terminal/legacy/keybindings/" \
  paste "'disabled'"
```

Terminal paste then needs `Shift+Insert` or middle-click. To undo:

```sh
gsettings reset \
  "org.gnome.Terminal.Legacy.Keybindings:/org/gnome/terminal/legacy/keybindings/" \
  paste
```

Skip this step on other terminals, but check whether they bind the key
themselves.

### 4. Start tmux fresh

```sh
tmux kill-server 2>/dev/null; tmux
```

A restart is needed rather than a reload: tmux negotiates extended-key support
with the terminal when a client attaches, so `extended-keys on` does not take
effect on an already-connected client.

### 5. Verify

Scroll up with the wheel, drag-select text, press `Ctrl+Shift+V`, then paste
elsewhere. If nothing arrives, confirm `xclip` is on `PATH` and that the
terminal actually emits the key — `extended-keys` depends on terminal support
and is not universal.

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
  to always enter copy-mode. Cost: applications no longer receive wheel events
  for their own scrolling.
- A pane in the alternate screen buffer keeps no scrollback at all, so no
  binding can scroll back through a full-screen application's output. Only
  `alternate-screen off` changes that, at the cost of leaving such an
  application's screen behind in history when it exits.

## Key bindings added

| Key | Where | Does |
|---|---|---|
| Wheel up | any pane | enter copy-mode and scroll back |
| Drag-select | copy-mode | select and copy, staying in copy-mode |
| `Ctrl+Shift+V` | copy-mode | copy selection to the system clipboard |
| `q` | copy-mode | exit, returning to the live prompt |
| `prefix` + `r` | anywhere | reload this config |
