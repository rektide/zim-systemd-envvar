# zim-systemd-envvar

> [Zimfw](https://zimfw.sh/) module to sync shell environment variables into the systemd user manager.

Every new shell session checks whether whitelisted env vars have drifted from `systemctl --user show-environment` and imports only the ones that changed. An md5 hash cache in `$XDG_RUNTIME_DIR` makes the no-change case essentially free.

Only vars that are **set and exported** in the shell are imported. zsh keeps some parameters (e.g. `USERNAME`) set-but-unexported; `systemctl import-environment` reads the process environment, so such a var would warn and never converge — the sync skips them instead and reports them in **one batched line per group** (`systemd-envvar-sync: <group>: unset or unexported, skipped: …`), and only for groups that have at least one importable member, so groups for features you are not using stay silent. A shell whose importable values match the cache is a complete no-op.

## Variable groups

Variables are organized into named groups. Enable or disable groups, or override the vars in any group, all via `zstyle` in your `~/.zshrc` **before** the module loads.

| Group     | Default vars                                                                                                                                    |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| wayland   | `WAYLAND_DISPLAY` `DISPLAY`                                                                                                                     |
| niri      | `NIRI_SOCKET`                                                                                                                                   |
| dbus      | `DBUS_SESSION_BUS_ADDRESS`                                                                                                                      |
| xdg       | `XDG_RUNTIME_DIR` `XDG_CURRENT_DESKTOP` `XDG_SESSION_DESKTOP` `XDG_SESSION_TYPE` `XDG_SESSION_CLASS` `XDG_SEAT` `XDG_VTNR` `DESKTOP_SESSION` |
| gdm       | `GDMSESSION` `GDM_LANG` `DESKTOP_STARTUP_ID`                                                                                                    |
| auth      | `SSH_AUTH_SOCK` `GPG_AGENT_INFO` `XAUTHORITY`                                                                                                   |
| ui        | `XCURSOR_SIZE` `XCURSOR_THEME` `COLORTERM`                                                                                                      |
| kitty     | `KITTY_WINDOW_ID` `KITTY_PID` `KITTY_PUBLIC_KEY` `KITTY_LISTEN_ON` `KITTY_INSTALLATION_DIR` `TERMINFO`                                          |
| general   | `HOME` `LANG` `TZ` `PAGER` `TERM` `PATH` `SHELL` `USER` `LOGNAME`                                                                    |

Default enabled groups: **wayland niri dbus xdg gdm auth ui kitty general**.

## Configuration

All `zstyle` calls go in `~/.zshrc` before `zimfw` initialization.

| Context | Key | What | Default |
| ------- | --- | ---- | ------- |
| `:zim:systemd-envvar` | `groups` | Which named groups to enable | `wayland niri dbus xdg gdm auth ui kitty general` |
| `:zim:systemd-envvar:<group>` | `vars` | Override a group's var list | See groups table |
| `:zim:systemd-envvar` | `extra-vars` | Ad-hoc vars outside any group | _(none)_ |
| `':zim:systemd-envvar' groups wayland dbus xdg gdm auth ui kitty general` | | Enable all groups | |
| `':zim:systemd-envvar:wayland' vars WAYLAND_DISPLAY DISPLAY` | | Override wayland vars (drop `NIRI_SOCKET`) | |
| `':zim:systemd-envvar' extra-vars VAULT_ADDR KUBECONFIG` | | Add custom vars | |

## Manual sync

The `systemd-envvar-sync` function is also available as an interactive command:

```zsh
systemd-envvar-sync
```

## Helper for other services: `bin/systemd-env-merge`

`systemctl [--user] set-environment KEY=VAL` **replaces** each variable, which
stomps on path-like vars (`PATH`, `PYTHONPATH`, …) that are meant to be
accumulated. `bin/systemd-env-merge` is a standalone bash helper any unit or
hook can call instead — it **merges** path-like vars (ordered union + dedup)
and replaces scalars. Shellcheck-clean; see `systemd-env-merge --help`.

Two patterns:

**Shape A — atomic (preferred for new units).** A service's `ExecStart` calls
`push` instead of `set-environment`:

```bash
mapfile -t kv < <(cd "$HOME" && mise env -J | jq -r 'to_entries[]|"\(.key)=\(.value)"')
systemd-env-merge push --pathlike PATH -- "${kv[@]}"
```

**Shape B — retrofit a unit that already stomps.** Snapshot before, merge after:

```ini
ExecStartPre =systemd-env-merge snapshot -o %t/%n.snap PATH
ExecStart    =<the existing set-environment>
ExecStartPost=systemd-env-merge restore --pathlike PATH %t/%n.snap
```

`push`/`restore` precedence: `push` puts new entries **first** (caller wins);
`restore` keeps current manager entries first and appends back any snapshot
entries that went missing (so a stomp becomes an append). Both skip unchanged
vars and no-op when there is nothing to do.

## Install

Add to your `~/.zimrc`:

```
zmodule rektide/zim-systemd-envvar
```
