# agentbox

Run coding agents (Claude Code, pi, Cursor CLI) in throwaway containers, each
with its own Docker-in-Docker sidecar. Only the current project directory is
mounted; `$HOME` never is.

## Install

```sh
brew install ouzman/tap/agentbox
```

Or put `agentbox` and the `b*` symlinks somewhere on your `PATH`. Requires
bash and a `docker` CLI (Docker Desktop, colima, podman-docker, ...).

## Usage

```sh
agentbox [-n name] claude|pi|cursor-agent [agent args...]
agentbox build                 # (re)build the base image
agentbox ls                    # list running sessions
agentbox shell [name] [cmd...] # shell into a session (default: the one in $PWD)
AGENTBOX_UPDATE=1 agentbox claude   # force an agent update now
AGENTBOX_IMAGE=myimage agentbox pi  # use a different image tag
```

Shortcuts (symlinks to `agentbox`, busybox-style):

| command         | same as                  |
|-----------------|--------------------------|
| `bclaude`       | `agentbox claude`        |
| `bpi`           | `agentbox pi`            |
| `bcursor-agent` | `agentbox cursor-agent`  |

A leading `-n name` still names the session: `bclaude -n foo` is
`agentbox -n foo claude`.

## Claude Code status line

The image ships [ccusage](https://github.com/ryoppippi/ccusage) and an
`agentbox-statusline` wrapper that prints two lines:

```
🤖 Opus 5.5 | 💰 $1.23 session / ... | 🧠 62,728 (31%)   <- ccusage statusline
📁 myproj | 🌿 feat/x | 🌳 feat-x | [ab-myproj-123]      <- git + agentbox session
```

The worktree part only appears in a linked worktree.
Claude inside agentbox uses `~/.agentbox/claude` as its config dir, so add this
to `~/.agentbox/claude/settings.json` (extra args go to `ccusage statusline`):

```json
"statusLine": { "type": "command", "command": "agentbox-statusline", "padding": 0 }
```
