# ctt — Cyrus' tmux TUI

A small, dependency-light terminal UI for running plain terminals and coding
agents ([Claude Code](https://github.com/anthropics/claude-code) / Codex)
inside named [tmux](https://github.com/tmux/tmux) sessions. Launch a new shell
or agent in any directory, jump back into a running session, or kill it off —
all from one arrow-driven screen.

It's a single Bash script. No runtime beyond tmux.

```
  ctt ❯ tmux sessions
  2 session(s)

   ＋  New session
   terminal-api-0603      2m ago     ~/work/api
   codex-web-ui-0603      attached   ~/dd/web-ui

  ↑/↓ move · ⏎ open · n new · d delete · r refresh · q quit
```

## Features

- **One cohesive TUI** — full-screen, raw-mode, with a highlight bar. Arrow
  keys (or `k`/`j`) everywhere; `→`/`Enter` to act, `←`/`Esc` to go back.
- **Session list** — shows each session's last activity (`2m ago`, `attached`)
  and working directory, plus a pinned *New session* row.
- **Plain terminals by default** — the first launcher is a regular tmux shell,
  so `ctt` works even when no coding-agent CLI is installed.
- **Guided new-session flow** — pick a launcher, choose a working directory,
  then name the session. The default name is built from the launcher, the folder,
  and the date/time (e.g. `terminal-web-ui-0529-1430`) and is fully editable.
- **Configurable launchers** — override the default Claude/Codex flags, reorder
  launchers, or add your own commands from `~/.config/ctt/config.bash`.
- **Directory picker** — browse the filesystem with arrows; just start typing
  to filter the current directory (case-insensitive). Symlinked directories are
  enterable and marked with `→`.
- **Conflict & delete handling** — attach / kill+recreate when a name exists;
  confirm before deleting a session.
- **Clean terminal handling** — restores cursor and terminal state on quit,
  `Ctrl-C`, or error.

## Requirements

- `bash` 4+
- `tmux`

## Install

```sh
curl -fsSL https://raw.githubusercontent.com/ckrowbar/ctt/main/ctt \
  -o ~/.local/bin/ctt && chmod +x ~/.local/bin/ctt
```

Or clone and symlink:

```sh
git clone https://github.com/ckrowbar/ctt.git
ln -s "$PWD/ctt/ctt" ~/.local/bin/ctt   # ensure ~/.local/bin is on $PATH
```

Then just run:

```sh
ctt
```

## Updating

```sh
ctt --update
```

This replaces the installed script in place with the latest version from
`main`, after validating the download. It works wherever `ctt` lives on your
`PATH` (no need to remember the install path), and it preserves a symlinked
install by writing through the link.

If you installed via `git clone`, `ctt --update` detects the checkout and points
you at the right command instead:

```sh
git -C /path/to/ctt pull
```

Updating a system-wide install may need elevated permissions: `sudo ctt --update`.

## Keybindings

| Screen           | Keys                                                          |
|------------------|---------------------------------------------------------------|
| Session list     | `↑/↓` `k/j` move · `⏎`/`→` open · `n` new · `d` delete · `r` refresh · `q` quit |
| Launcher / menus | `↑/↓` move · `⏎` select · `Esc` cancel                        |
| Name input       | type to edit · `⏎` accept · `⌫` delete · `^U` clear · `Esc` cancel |
| Directory picker | `↑/↓` move · `→`/`⏎` open · `←` parent · type to filter · `⌫` del filter · `Esc` clear/cancel |

`Ctrl-C` exits cleanly from any screen.

## How it works

`ctt` launches sessions detached and then attaches:

```sh
tmux new-session -d -s "$name" -c "$workdir"
tmux attach -t "$name"
```

Sessions also get `set-titles` enabled so the session name is published via the
terminal title (handy for IDE terminal sidebars).

The whole thing is one file (`ctt`) with a few shared TUI primitives
(`read_key`, `draw`, `tui_select`, `tui_input`, `tui_confirm`) on top of a
single raw-mode session bound to `/dev/tty`.

## Customizing Launchers

By default, launchers are shown in this order:

```sh
terminal
claude
codex
pi
opencode
```

`terminal` starts tmux's default shell. The agent defaults are:

```sh
claude --permission-mode auto
codex --sandbox workspace-write --ask-for-approval on-request
pi
opencode
```

Create `~/.config/ctt/config.bash` to customize them:

```bash
# Reorder or limit the launchers shown in the picker.
CTT_LAUNCHER_IDS=(terminal codex claude)

# Optional display labels.
CTT_LAUNCHER_LABELS[codex]="codex (full auto)"

# Commands are Bash arrays, so quoted arguments stay intact.
CTT_CMD_CODEX=(codex --sandbox workspace-write --ask-for-approval never)
CTT_CMD_CLAUDE=(claude --permission-mode plan)
```

You can add your own launcher by adding an id and defining the matching command
array:

```bash
CTT_LAUNCHER_IDS=(terminal nvim codex)
CTT_LAUNCHER_LABELS[nvim]="nvim"
CTT_CMD_NVIM=(nvim)
```

Launcher ids must start with a letter and contain only letters, digits, and
underscores. An empty command array launches tmux's default shell:

```bash
CTT_CMD_TERMINAL=()
```

Set `CTT_CONFIG=/path/to/config.bash` to load a different config file.
