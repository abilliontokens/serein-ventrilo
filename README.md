# Ventrilo Mode for Serein

> **Preview SDK, not yet released.** This plugin needs a Serein build with the
> `interface_ui` capability from [Serein PR #1042](https://github.com/ViceVerse-cz/Serein/pull/1042).
> Released clients do not have it yet.

A [Serein](https://github.com/ViceVerse-cz/Serein) extension that replaces the
navigation, conversation and member panes with a slim, Ventrilo-style tree: the
picked server's text and voice channels and the people in each voice channel.
Your servers appear as small icons at the left of the bottom control row, next to
the client's own add-a-server button, and the gear, mute/deafen and disconnect
controls stay on that row. One native window can show beside the tree: the call
stage or one text channel's chat. The plugin draws nothing itself. It restates a
declarative interface on every result, and Serein renders it generically.

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
| `voice_connect` | Propose joining a voice channel when you double-click one. |
| `voice_control` | Keep the call-control lane available; today the host's bottom strip already provides mute, deafen and disconnect. |
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
- **Channel rows** follow the order of the client's channel list. Text channels
  (server text and announcement channels) carry a `#` icon; voice channels a
  speaker. Categories, threads, forums and stage channels are not listed.
- **Text-channel rows**: a double-click opens that channel's chat beside the tree
  and highlights the row; double-clicking it again closes the chat. A single
  click does nothing. Opening a text channel never proposes anything.
- **Voice-channel rows**: a single click does nothing. A double-click proposes
  joining it (`join_voice`; Serein still shows
  the proposal and its **Apply** button) and shows the call window beside the
  tree. Double-clicking the channel you are already in closes the call window
  (or brings it back) — the call itself stays connected.
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

The SDK dependency is pinned in `Cargo.toml` to the Serein revision that adds
`interface_ui`.

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
