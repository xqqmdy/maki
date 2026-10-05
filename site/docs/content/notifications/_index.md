+++
title = "Notifications"
weight = 9
[extra]
group = "Reference"
+++

# Notifications

Maki can tell you when a session finishes or needs your input. This is useful
when you move to another terminal while Maki works.

Notifications are enabled by default. A prompt that waits on you always
notifies, because the agent stays parked until you answer. `Agent turn
complete` is skipped only when Maki can tell you are watching: the terminal
reported focus, or you typed in the last 30 seconds.

Maki uses these messages:

- `Agent turn complete` or a preview of the response, up to 200 characters.
- `Permission requested: <tool>` for a permission prompt.
- `Authentication required` when authentication needs attention.
- `Question requested` for a question prompt.
- `Plan ready` when a plan is ready.

Response previews can appear in your operating system's notification history.
Maki does not include tool arguments, permission scopes, question bodies, plan
content, or error details. Use `bell` for a message-free alert, or use `off`
to disable notifications if response text should not reach notification
history.

## Configuration

Set `ui.notifications` in `~/.config/maki/init.lua`:

```lua
maki.setup({
  ui = {
    notifications = "auto",
  },
})
```

| Value | Behavior |
| --- | --- |
| `auto` | Use OSC 9 in a supported terminal, OSC 777 in Windows Terminal, and BEL otherwise. |
| `osc9` | Always send an OSC 9 notification. |
| `osc777` | Always send an OSC 777 notification. |
| `bell` | Always send the terminal bell. |
| `off` | Do not send notifications. |

`auto` supports Ghostty, iTerm2, Kitty, Warp, and WezTerm. An unknown terminal
uses BEL. Your terminal settings decide whether BEL makes a sound or shows a
visual alert.

Maki also recognizes `xterm-ghostty` and `xterm-kitty` from `TERM`. This lets
OSC 9 work when an SSH connection does not preserve `TERM_PROGRAM`.

## tmux

OSC 9 and OSC 777 need passthrough:

```tmux
set -g allow-passthrough all
```

Use `allow-passthrough all`, not `allow-passthrough on`. The `on` value permits
passthrough only while the Maki pane is visible. tmux drops the notification
after you change to another tmux window.

Focus events are a separate setting:

```tmux
set -g focus-events on
```

This lets Maki suppress a turn completion you are already watching. Without it
Maki falls back to your last keypress and notifies for anything slower than 30
seconds.

Add the settings to `~/.tmux.conf`, then reload the file or restart tmux.

## Other terminal multiplexers

Maki wraps OSC 9 for GNU screen. GNU screen does not pass terminal focus
events to Maki, so Maki does not suppress notifications there. A notification
can appear while the GNU screen window has focus.

Maki sends OSC 9 directly through Zellij.

## Windows Terminal

Windows Terminal does not support OSC 9, so `auto` sends OSC 777 there. Desktop
notifications need Windows Terminal 1.26 Preview or newer with notifications
enabled in Settings, and the window must be in the background. Older versions
drop OSC 777 silently. BEL stays the fallback for terminals that match no
notification protocol.

Inside a multiplexer such as herdr or tmux, the multiplexer's pane parser
consumes OSC 777 before it reaches Windows Terminal. Run the multiplexer's own
notification feature instead; herdr forwards agent state changes as OSC 777
toasts.

## Focus on Windows

This terminal focus protocol is not available on Windows. Maki treats the
terminal as unfocused so an explicit `bell`, `osc9`, or `osc777` setting still
works.
