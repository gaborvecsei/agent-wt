# agent-wt (`wt`)

One git worktree and one tmux session per branch, so every coding agent works in its own checkout. Create, attach to, and remove them from a single fzf picker.

## Requirements

- `bash` (3.2 or newer, so the macOS default works), `git`, and `tmux`
- `fzf` for the picker
- [`ma`](https://github.com/gaborvecsei/my-agents) and `jq` for the agent status columns (optional)

## Setup

```sh
curl -fsSL --create-dirs https://raw.githubusercontent.com/gaborvecsei/agent-wt/main/wt -o ~/.local/bin/wt
chmod +x ~/.local/bin/wt
```

Any directory on your `PATH` works.

`wt` expects the clone in a folder named `main`, with the worktrees next to it:

```
repo/
├── .wt.conf   # wt config, not part of the repository
├── main/      # the clone
└── feat-x/    # worktree for branch feat/x, tmux session repo-feat-x
```

`wt init` creates this layout:

```sh
wt init git@github.com:you/repo.git   # clone into repo/main
wt init ~/code/repo                   # move an existing clone to ~/code/repo/main (asks first)
```

## Usage

Run these from `repo/`, `main/`, or any worktree:

```sh
wt new feat/login               # worktree + tmux session, then attach. New branches start from origin's default branch
wt new fix/bug --from v1.2      # start the new branch from another ref
wt                              # picker: enter attaches, ctrl-x removes, esc cancels
wt resume feat-login            # attach without the picker (alias: re)
wt ls                           # list worktrees
wt rm feat-login -b             # remove the worktree and its session, -b also deletes the branch
```

`wt rm` refuses to remove a worktree with uncommitted changes or unpushed commits. Add `--force` to remove it anyway. A squash-merged branch whose remote branch was deleted counts as unpushed, so it also needs `--force`.

## Config

`repo/.wt.conf` is plain bash. `wt init` writes a commented template.

```bash
WT_PREFIX="proj"                             # tmux session prefix (default: the folder name)
WT_COPY=(.env .claude/settings.local.json)   # ignored files copied from main/ into each new worktree
WT_POST_CREATE="uv sync"                     # command run inside each new worktree
```

## Agent status

If `ma` and `jq` are on your `PATH`, the picker shows the Claude Code and pi agents running in each worktree. `●` is waiting for input, `▶` is busy, `○` is idle, and `?` is unknown (pi outside tmux). Without `ma` or `jq`, `wt` hides these columns.

## tmux

Add to `~/.tmux.conf` to open the worktree picker with `prefix + W` and the `ma` agent picker with `prefix + a`:

```tmux
# Keep the popup open on failure, otherwise the error scrolls past as it closes
bind W display-popup -E -w 90% -h 70% -d "#{pane_current_path}" "wt || { printf '[enter] '; read _; }"
bind a display-popup -E -w 90% -h 70% "ma || { printf '[enter] '; read _; }"
```

If the tmux server doesn't see your `PATH`, use the full path, such as `~/.local/bin/wt`. Reload with `tmux source ~/.tmux.conf`.

## License

[MIT](LICENSE)
