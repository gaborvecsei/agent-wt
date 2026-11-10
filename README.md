# agent-wt (`wt`)

## Requirements

- `git`
- `tmux`
- `fzf` for the picker
- Optional, but HIGHLY recommended: [`ma`](https://github.com/gaborvecsei/my-agents) and `jq`, for the agent status columns in the picker

## Install

```sh
curl -fsSL --create-dirs https://raw.githubusercontent.com/gaborvecsei/agent-wt/main/wt -o ~/.local/bin/wt
chmod +x ~/.local/bin/wt
```

Any directory on your `PATH` works.

Then set up a repository in the layout above. `wt init` clones into `repo/main`, or moves an existing clone there, and writes a commented `.wt.conf`:

```sh
wt init ~/code/repo                   # move an existing clone to ~/code/repo/main (asks first)
wt init git@github.com:you/repo.git   # clone into repo/main
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
- `*` if the worktree has uncommitted changes
- `+N/-M` for commits ahead of and behind the upstream branch
- one glyph per Claude Code and pi agent in the worktree: `●` waiting for input, `▶` busy, `○` idle, `?` unknown. These columns need `ma` and `jq`.

`wt rm` won't touch a worktree with uncommitted changes or unpushed commits unless you pass `--force`. Squash-merged branches with a deleted remote count as unpushed too.

## Config

This is a bit of help, but completely optional.

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
