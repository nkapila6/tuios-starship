# tuios-starship

[starship](https://starship.rs) pills for [tuios](https://tuios.dev). They show what tuios is doing without you having to look at it, and they stay out of the way when there's nothing to say.

```
 tuios 2 detached          outside tuios: sessions running in the background
 tuios session-0 1/9       inside a pane: your session and workspace
 󰚩 1 waiting 2 done 1 msg    anywhere: agents that need you
```

Every pill hides itself when it has nothing to show, so most of the time you'll see one or none.

## Setup

You need `tuios`, `jq` and a Nerd Font (for the rounded edges and the robot icon).

1. Copy the blocks you want from [`tuios.toml`](tuios.toml) into `~/.config/starship.toml`.
2. Add them to `format` or `right_format`:

```toml
right_format = """${custom.tuios_agents}${custom.tuios}${custom.tuios_here}$time"""
```

## The pills

### `custom.tuios`: sessions, outside tuios

```sh
tuios ls --json | jq '[.[] | select(.saved | not)] | ...'
```

- Counts live sessions. When the daemon is stopped, `tuios ls` lists sessions saved on disk with `"saved": true`, so those get filtered out.
- Adds ` detached` when no session has a client attached. That's the "you left something running" reminder.
- Only shows outside tuios. tuios sets `$TUIOS_SESSION` and `$TUIOS_PANE_ID` in every pane, and the `when` check skips the module (and the command) if either is set. Inside tuios you can already see your sessions.

### `custom.tuios_here`: where you are, inside tuios

```sh
tuios session-info -s "$TUIOS_SESSION" --json
```

Shows the session name from `$TUIOS_SESSION` plus `current_workspace/num_workspaces`. Handy once you have a few sessions and lose track of which pane belongs where.

### `custom.tuios_agents`: agents waiting on you

```sh
tuios list-agents --json --all-sessions
```

Looks at every agent pane across all sessions and builds up to three counts:

- `waiting`: agents with `needs_you` set, i.e. blocked on a prompt or permission.
- `done`: agents with `finished_unread`, i.e. they finished and you haven't looked yet.
- `msg`: `human_unread`, messages agents left in your inbox.

Zero counts are dropped, and if all three are zero the module prints nothing, so the pill only appears when something actually wants you. It works from any shell, so a plain terminal outside tuios still tells you when an agent got stuck.

## How hiding works

Each `format` is wrapped in `( ... )`. In starship that means "only render this if a variable inside has a value". The jq filters print `empty` instead of `0`, so no output means no pill and you never see an empty bubble.

## Cost

These commands talk to the already-running daemon, about 10-15ms each. `tuios` and `tuios_here` are mutually exclusive, so you pay for two commands per prompt at most. Set `command_timeout` at the top of your starship config if you want a hard cap.

## Tweaks

- **Colours:** purple is the tuios logo, red is for attention. If you use a starship `palette`, swap the hex values for palette names.
- **Muted look:** there's a commented grey-pill format at the bottom of `tuios.toml`.
- **Remote hosts:** `tuios ls` and `list-agents` both take `--all-hosts`. They're left out because an unreachable host would slow every prompt.
