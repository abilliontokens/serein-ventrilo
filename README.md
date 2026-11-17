# Ventrilo Mode for Serein

> **Preview SDK, not yet released.** This plugin needs a Serein build with the
> `interface_ui` capability from [Serein PR #1042](https://github.com/ViceVerse-cz/Serein/pull/1042).
> Released clients do not have it yet.

A [Serein](https://github.com/ViceVerse-cz/Serein) extension that replaces the
navigation, conversation and member panes with a slim, Ventrilo-style tree: the
picked server's categories, text and voice channels and the people in each voice
channel. While the layout shows, it asks for a slim 340 × 640 window. Your servers
appear as small icons at the left of the bottom control row, next to the client's
own add-a-server button, and the plugin places the disconnect, mute, deafen and
settings controls at the right of that row. One native window can show beside the
tree: the call stage or one text channel's chat. The plugin draws nothing itself.
It restates a declarative interface on every result, and Serein renders it
generically.

It doubles as a reference for building on `interface_ui`.

## Grants

| Capability | Why |
| --- | --- |
| `interface_ui` | Replace the main panes with the plugin's tree and choose the window beside it. |
| `storage` | Remember the on/off choice, picked server and the open window. |
| `guild_directory` | List your servers as dock icons in the bottom row. |
| `channel_directory` | List each server's text and voice channels. |
| `voice_state` | Read the current call and the voice roster: who sits in which voice channel, with names, mute/deafen (self and server) and speaking flags. |
| `app_context` | Recognize your own row for the `You` badge. |
| `voice_connect` | Let the host join a voice channel directly when you double-click its row (the row's native join; no Apply step). |
| `voice_control` | Keep the call-control lane available; the disconnect, mute and deafen buttons the plugin places on the bottom row are the host's own. |
| `app_events` | Refresh the tree when voice state changes (`voice` events), so speaking and mute indicators update. |
| `data_events` | Refresh the tree when servers or channels change (`channels` events). |

## Behavior

- **Restore Ventrilo layout** (activation) re-applies the saved choice whenever
  the plugin is enabled or Serein starts: on by default, with the first server
  in directory order picked.
- **Server icons** in the bottom row switch the tree to that server without
  joining or leaving a call; the picked server is highlighted, and its name heads
  the tree. The **+** after them is Serein's own add-a-server button. The plugin
  never sees what it does.
- **Channel rows** follow Discord's sidebar order: channels outside any category
  first, then each category by position (ties by ID) followed by its channels.
  Within each group text channels come before voice channels, then by position,
  then by ID. Text channels (server text and announcement channels) carry a `#`
  icon; voice channels a speaker. Threads, forums and stage channels are not
  listed.
- **Category rows** show each category's name as a plain, unclickable heading
  without an icon. A category with no listed channel is left out, and a channel
  whose category the client has not loaded counts as uncategorized.
- **Text-channel rows**: a double-click opens that channel's chat beside the tree
  and highlights the row; double-clicking it again closes the chat. A single
  click does nothing.
- **Voice-channel rows**: a single click does nothing. A double-click (or Enter
  on the focused row) joins the channel directly. Serein runs its own join, with
  your current mute and deafen state and its usual call-switch confirmation,
  and labels the row with the channel's real name. The plugin shows the call
  window beside the tree. Double-clicking the channel you are already in closes
  the call window (or brings it back); the call itself stays connected.
- **Bottom row controls**: disconnect (only during a call), mute, deafen and
  settings, left to right.
- **Window size**: while the layout shows, Serein resizes the window to a slim
  340 × 640 (centered, clamped to the monitor) and restores the previous size
  when the layout ends.
- **One window at a time**: opening a chat replaces the call window and opening
  the call replaces a chat.
- **People rows** appear under every voice channel of the picked server that the
  voice roster places people in, one row per person, with the status icon left
  of the name: headphones-slash while deafened (self or server), microphone-slash
  while muted (self or server), a speaker otherwise. The icon is tinted danger
  while muted or deafened, positive while speaking, and muted otherwise. Your own
  row carries the `You` badge. People rows are not clickable.
- **Ventrilo Mode settings** describes the current state with a button that
  flips the layout; **Toggle Ventrilo layout** does the same from the panel
  list. Toggling off restores the normal layout immediately and is remembered
  for the next activation, together with the picked server and the open window.

With nothing saved, the layout is on. Fields this version no longer uses (such as
an older `folded` list) are ignored. If a field it does use has the wrong type
(for example an older boolean `window`), the plugin falls back to the same default
and leaves storage unchanged; the next change overwrites it with a valid value.

## Limitations

- A person's name can be missing until the client has loaded their member data;
  until then the row shows their user ID.
- The roster covers what the client currently knows; servers it has not loaded
  show their channels without people.
- Plugins see at most 100 loaded channels and 64 dock items. Servers past
  that are not shown.

## Build and install

The SDK dependency in `Cargo.toml` is pinned to Serein commit `0434bc7c`, the
preview revision with native row actions, plugin-chosen controls and window size.

```sh
rustup target add wasm32-unknown-unknown
cargo test --locked
cargo build --locked --release --target wasm32-unknown-unknown
python pack.py manifest.json target/wasm32-unknown-unknown/release/ventrilo.wasm packages/ventrilo.serein-extension
```

`pack.py` is Serein's packaging script (`extensions/pack.py`), copied unchanged.
A prebuilt package is committed at `packages/ventrilo.serein-extension`.

Import it in **Settings > Extensions**, review and grant the capabilities above,
then enable the plugin.

## License

MIT OR Apache-2.0, matching Serein.
