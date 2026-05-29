# cct — Cyrus' tmux TUI

A small, dependency-light terminal UI for running coding agents
([Claude Code](https://github.com/anthropics/claude-code) / Codex) inside named
[tmux](https://github.com/tmux/tmux) sessions. Launch a new agent in any
directory, jump back into a running one, or kill it off — all from one
arrow-driven screen.

It's a single Bash script. No runtime beyond tmux and a coding-agent CLI.

```
  cct ❯ coding-agent sessions
  2 session(s)

   ＋  New session
   claude-1430            2m ago     ~/work/api
   codex-0915             attached   ~/dd/web-ui

  ↑/↓ move · ⏎ open · n new · d delete · r refresh · q quit
```

## Features

- **One cohesive TUI** — full-screen, raw-mode, with a highlight bar. Arrow
  keys (or `k`/`j`) everywhere; `→`/`Enter` to act, `←`/`Esc` to go back.
- **Session list** — shows each session's last activity (`2m ago`, `attached`)
  and working directory, plus a pinned *New session* row.
- **Guided new-session flow** — pick a provider, name the session (editable,
  with a sensible `provider-HHMM` default), then choose a working directory.
- **Directory picker** — browse the filesystem with arrows; symlinked
  directories are enterable and marked with `→`.
- **Conflict & delete handling** — attach / kill+recreate when a name exists;
  confirm before deleting a session.
- **Clean terminal handling** — restores cursor and terminal state on quit,
  `Ctrl-C`, or error.

## Requirements

- `bash` 4+
- `tmux`
- GNU `coreutils` / `findutils` (`find -printf`, `-xtype`; `date`) — i.e. Linux
  or `brew install coreutils findutils` on macOS
- At least one agent CLI on your `PATH`: `claude` and/or `codex`

## Install

```sh
curl -fsSL https://raw.githubusercontent.com/ckrowbar/cct/main/cct \
  -o ~/.local/bin/cct && chmod +x ~/.local/bin/cct
```

Or clone and symlink:

```sh
git clone https://github.com/ckrowbar/cct.git
ln -s "$PWD/cct/cct" ~/.local/bin/cct   # ensure ~/.local/bin is on $PATH
```

Then just run:

```sh
cct
```

## Updating

```sh
cct --update
```

This replaces the installed script in place with the latest version from
`main`, after validating the download. It works wherever `cct` lives on your
`PATH` (no need to remember the install path), and it preserves a symlinked
install by writing through the link.

If you installed via `git clone`, `cct --update` detects the checkout and points
you at the right command instead:

```sh
git -C /path/to/cct pull
```

Updating a system-wide install may need elevated permissions: `sudo cct --update`.

## Keybindings

| Screen           | Keys                                                          |
|------------------|---------------------------------------------------------------|
| Session list     | `↑/↓` `k/j` move · `⏎`/`→` open · `n` new · `d` delete · `r` refresh · `q` quit |
| Provider / menus | `↑/↓` move · `⏎` select · `Esc` cancel                        |
| Name input       | type to edit · `⏎` accept · `⌫` delete · `^U` clear · `Esc` cancel |
| Directory picker | `↑/↓` move · `→`/`⏎` open · `←` parent · `~` home · `q` cancel |

`Ctrl-C` exits cleanly from any screen.

## How it works

`cct` launches agents detached and then attaches:

```sh
tmux new-session -d -s "$name" -c "$workdir" claude --permission-mode auto
tmux attach -t "$name"
```

Sessions also get `set-titles` enabled so the session name is published via the
terminal title (handy for IDE terminal sidebars). Agents run in "auto" mode —
in-workspace edits are auto-approved, riskier actions still prompt.

The whole thing is one file (`cct`) with a few shared TUI primitives
(`read_key`, `draw`, `tui_select`, `tui_input`, `tui_confirm`) on top of a
single raw-mode session bound to `/dev/tty`.

## Customizing

The agent commands live in `screen_new()`:

```sh
case "$provider" in
  claude) cmd=(claude --permission-mode auto) ;;
  codex)  cmd=(codex  --sandbox workspace-write --ask-for-approval on-request) ;;
esac
```

Edit those to change flags or add providers.
