# tuios-starship

A tiny [starship](https://starship.rs) module that shows how many [tuios](https://tuios.dev) sessions you have running, right in your prompt.

```
 tuios 1           one live session, you're attached to it
 tuios 2 detached  sessions running with nothing attached
                   (nothing) daemon is down or there are no sessions
```

## Setup

You need `tuios`, `jq` and a Nerd Font (only for the rounded pill edges).

1. Copy the `[custom.tuios]` block from [`tuios.toml`](tuios.toml) into `~/.config/starship.toml`.
2. Add `${custom.tuios}` wherever you want it in `format` or `right_format`:

```toml
right_format = """${custom.tuios}$cmd_duration$time"""
```

That's it. Open a new prompt and it shows up.

## How it works

Starship's `custom` modules run a shell command on every prompt and print whatever it outputs. This one runs:

```sh
tuios ls --json 2>/dev/null | jq -r '
  [.[] | select(.saved | not)]
  | if length > 0
    then (length | tostring) + (if any(.attached) then "" else " detached" end)
    else empty end'
```

Step by step:

- `tuios ls --json` asks the tuios daemon for its sessions (name, window count, whether a client is attached, ...). Errors go to `/dev/null`, so a missing daemon just means no output.
- When the daemon is stopped, `tuios ls` lists sessions saved on disk with `"saved": true`. Those aren't running, so `select(.saved | not)` drops them.
- If there's at least one live session, it prints the count. If none of them has a client attached it adds ` detached`, which is a handy reminder that something is still running in the background.
- With zero sessions it prints nothing (`empty`).

The `format` is wrapped in `( ... )`, which in starship means "only render if there's a value inside". No output, no pill. You never see an empty bubble.

## Cost

`tuios ls` just talks to the already-running daemon, so it's around 10ms per prompt. If you want a hard ceiling, set `command_timeout` at the top level of your starship config.

## Tweaks

- **Colour:** the hex values in `format` are tuios' logo purple. If you use a starship `palette`, swap them for your palette names (`fg:tuios bg:surface` etc).
- **Muted look:** there's a commented grey-pill version in `tuios.toml` if filled pills are too loud.
- **Remote hosts:** `tuios ls --all-hosts` also counts sessions on machines in your `[hosts]` config. It's left out on purpose since an unreachable host would slow every prompt.

## Limits

It counts sessions, it doesn't know which one you're in. tuios doesn't expose the session name to the shell, so there's nothing to show.
