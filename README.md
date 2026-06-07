# zim-systemd-envvar

> [Zimfw](https://zimfw.sh/) module to sync shell environment variables into the systemd user manager.

Every new shell session checks whether whitelisted env vars have drifted from `systemctl --user show-environment` and imports only the ones that changed. An md5 hash cache in `$XDG_RUNTIME_DIR` makes the no-change case essentially free.

## Variable groups

Variables are organized into named groups. Enable or disable groups, or override the vars in any group, all via `zstyle` in your `~/.zshrc` **before** the module loads.

| Group     | Default vars                                                                                                                                    |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| wayland   | `WAYLAND_DISPLAY` `DISPLAY` `NIRI_SOCKET`                                                                                                       |
| dbus      | `DBUS_SESSION_BUS_ADDRESS`                                                                                                                      |
| xdg       | `XDG_RUNTIME_DIR` `XDG_CURRENT_DESKTOP` `XDG_SESSION_DESKTOP` `XDG_SESSION_TYPE` `XDG_SESSION_CLASS` `XDG_SEAT` `XDG_VTNR` `DESKTOP_SESSION` |
| gdm       | `GDMSESSION` `GDM_LANG` `DESKTOP_STARTUP_ID`                                                                                                    |
| auth      | `SSH_AUTH_SOCK` `GPG_AGENT_INFO` `XAUTHORITY`                                                                                                   |
| ui        | `XCURSOR_SIZE` `XCURSOR_THEME` `COLORTERM`                                                                                                      |
| kitty     | `KITTY_WINDOW_ID` `KITTY_PID` `KITTY_PUBLIC_KEY` `KITTY_LISTEN_ON` `KITTY_INSTALLATION_DIR` `TERMINFO`                                          |
| general   | `HOME` `LANG` `TZ` `PAGER` `TERM` `PATH` `SHELL` `USER` `USERNAME` `LOGNAME`                                                                    |

Default enabled groups: **wayland dbus xdg auth ui general** (`gdm` and `kitty` available but not enabled).

## Configuration

All `zstyle` calls go in `~/.zshrc` before `zimfw` initialization.

| Context | Key | What | Default |
| ------- | --- | ---- | ------- |
| `:zim:systemd-envvar` | `groups` | Which named groups to enable | `wayland dbus xdg auth ui general` |
| `:zim:systemd-envvar:<group>` | `vars` | Override a group's var list | See table above |
| `:zim:systemd-envvar` | `extra-vars` | Ad-hoc vars outside any group | _(none)_ |

```zsh
# enable all groups
zstyle ':zim:systemd-envvar' groups wayland dbus xdg gdm auth ui kitty general

# override wayland group (e.g. no niri socket)
zstyle ':zim:systemd-envvar:wayland' vars WAYLAND_DISPLAY DISPLAY

# add custom vars
zstyle ':zim:systemd-envvar' extra-vars VAULT_ADDR KUBECONFIG
```

## Manual sync

The `systemd-envvar-sync` function is also available as an interactive command:

```zsh
systemd-envvar-sync
```

## Install

Add to your `~/.zimrc`:

```
zmodule rektide/zim-systemd-envvar
```
