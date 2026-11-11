# agent-wt (`wt`)

Tiny git worktree + tmux helper for managing AI coding agents in parallel.

![wt demo](demo/demo.gif)

> why?

Because I was tired of writing repetative commands over and over again.
And I was also tired of UI heavy bloated solutions packed with unnecessary features. I wanted something that just works, fast, and is easy to use.

## Requirements

- `git`
- `tmux`
- `fzf` for the picker
- Optional, but HIGHLY recommended as this allows the agent overview: [`ma`](https://github.com/gaborvecsei/my-agents) and `jq`, for the agent status columns in the picker

## Install

```sh
curl -fsSL --create-dirs https://raw.githubusercontent.com/gaborvecsei/agent-wt/main/wt -o ~/.local/bin/wt
chmod +x ~/.local/bin/wt
```

### First setup for a repo

```sh
wt init ~/code/repo                   # move an existing clone to ~/code/repo/main (asks first)
wt init git@github.com:you/repo.git   # clone into repo/main immediately
```

## Usage

Run these from `repo/`, `main/`, or any worktree, it just works:

```sh
wt new feat/login               # new worktree + tmux session, then attach (branches off origin's default branch)
wt new fix/bug --from v1.2      # branch off another ref
wt                              # picker: enter attaches, ctrl-x removes, esc cancels
wt resume feat-login            # attach without the picker (alias: re)
wt ls                           # list worktrees
wt rm feat-login -b             # remove worktree + session, -b also deletes the branch
```

Each row in the picker and in `wt ls` shows:

- `●` if the worktree's tmux session is running, `○` if it is not
- the worktree name and its branch
- the branch in yellow when it no longer matches the worktree name (switched inside the worktree)
- `*` if the worktree has uncommitted changes
- `+N/-M` for commits ahead of and behind the upstream branch
- one glyph per Claude Code and pi agent in the worktree: `●` waiting for input, `▶` busy, `○` idle, `?` unknown. These columns need `ma` and `jq`.

`wt rm` won't touch a worktree with uncommitted changes or unpushed commits unless you pass `--force`. Squash-merged branches with a deleted remote count as unpushed too. `-b` refuses if the branch was switched inside the worktree, since `wt` can't tell which branch you want gone.

Agents sometimes switch branches inside a worktree. The folder and tmux session stay put, and `wt resume`/`wt rm` still accept the original branch name. To make it rarer, tell your agent to stay on its branch (e.g. in `CLAUDE.md`).

## Config

This is just a bit of help, and completely optional.

`repo/.wt.conf`

```bash
WT_PREFIX="proj"                             # tmux session prefix (default: folder name)
WT_COPY=(.env .claude/settings.local.json)   # ignored files copied from main/ into new worktrees
WT_POST_CREATE="uv sync"                     # run inside each new worktree
```

## tmux key bindings

I bind the worktree picker to `prefix + W` and the `ma` agent picker to `prefix + a` in `~/.tmux.conf`:

```tmux
bind W display-popup -E -w 90% -h 70% -d "#{pane_current_path}" "wt || { printf '[enter] '; read _; }"
bind a display-popup -E -w 90% -h 70% "ma || { printf '[enter] '; read _; }"
```

The `|| { ...; read _; }` part keeps the popup open when the command fails, so you can read the error. If tmux can't find `wt`, use the full path (`~/.local/bin/wt`).
