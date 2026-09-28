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
agentbox build                 # force a rebuild of the base image
agentbox dockerfile            # print the base image's Dockerfile
agentbox ls                    # list running sessions
agentbox shell [name] [cmd...] # shell into a session (default: the one in $PWD)
AGENTBOX_UPDATE=1 agentbox claude   # force an agent update now
AGENTBOX_IMAGE=myimage agentbox pi  # use a different image tag
```

The base image is built on first use and rebuilt automatically when its
definition changes (e.g. after `brew upgrade agentbox`). Running sessions keep
the image they started with. A custom `AGENTBOX_IMAGE` is only built if missing,
never rebuilt.

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

## Published image

Tagging a release (`v*`) publishes the base image as
`ghcr.io/ouzman/agentbox:<tag>` (linux/amd64 + linux/arm64, user `dev` with
UID/GID 1000) for tools built on top of it, such as agentbox-k8s. It is the
same image `agentbox build` makes; `agentbox build` passes extra arguments to
`docker build`, and `AGENTBOX_UID`/`AGENTBOX_GID` override the host's IDs:

```sh
AGENTBOX_IMAGE=ghcr.io/ouzman/agentbox:v0.1.0 AGENTBOX_UID=1000 AGENTBOX_GID=1000 \
  agentbox build --platform linux/amd64,linux/arm64 --push
```

Every session starts through `/usr/local/bin/agentbox-boot` (baked into the
image): it installs or updates the agent named by `$AB_TOOL`, then execs its
arguments. A custom `AGENTBOX_IMAGE` built by an older agentbox lacks it;
rebuild it with `agentbox build`.
