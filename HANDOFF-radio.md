# HANDOFF: small radio widget (Radio 1 live / VRT NWS) for the XENEON EDGE

Owner: Christian (Belgium, listens in Dutch). Run in Claude Code on his Windows PC (iCUE + XENEON EDGE). Same workflow as the other handoffs: Corsair Skill + `icuewidget` CLI (see `HANDOFF.md` section 3 for the install steps).

## Why

- No radio widget exists on the Marketplace that plays VRT stations. iFrame and Web URL widgets need Medium size or larger, and VRT's web player will probably refuse to be embedded.
- A custom widget can sit in a **Small** slot (the smallest XENEON EDGE slot is 840x344 px per Corsair; the Skill has the other sizes) and play an audio stream directly.

## Goal

A small widget with:

- two station buttons: **Radio 1** (live stream) and **VRT NWS** (the continuous news stream, repeats bulletins),
- a large play/pause button,
- a status line: stopped, connecting, playing, or error,
- a volume slider (the Windows Volume widget exists too, but one in the widget is handy),
- touch-friendly targets (the Edge is a touchscreen).

## Stream addresses

Christian supplied these from VRT. Only the first is a complete address:

| Station | Address | Status |
|---|---|---|
| Radio 1 (MP3) | `http://icecast.vrtcdn.be/radio1-high.mp3` | complete, **use this** |
| Radio 1 (AAC) | `http://vrtcdn.be` | **incomplete** (host only, no mount path) |
| VRT NWS (MP3) | `https://vrtcdn.be` | **incomplete** (host only, no path) |

Do **not** guess the missing addresses. To find them:

1. Ask Christian for the full URLs first. He has the source list.
2. If he doesn't have them, an Icecast server normally lists its mount points on its status page. Try `http://icecast.vrtcdn.be/status-json.xsl` (this is a standard Icecast path, but it can be disabled, so it may fail). Pick the VRT NWS mount from the list and confirm it with a short test playback before using it.
3. If neither works, build the widget with Radio 1 only and leave VRT NWS configurable through a settings text field for the address, so he can paste it in later.

Use official VRT streams for personal listening only. No recording, re-streaming or scraping.

## Platform facts (from Elgato's docs)

- Widgets are HTML/JS rendered by QtWebEngine (Chromium 130). Manifest fields and the scaffold: use `icuewidget init` and copy the generated version fields instead of guessing.
- Permissions live in the manifest. Relevant types: `url` (network access: `{ "type": "url", "domain": "icecast.vrtcdn.be", "port": 80 }`) and `speaker` (`{ "type": "speaker" }`, permission to play audio output). Declare both, and test whether the `url` permission is needed for an `<audio>` element at all. Undeclared access causes a runtime Allow/Deny prompt.
- `interactive: true` is needed in the manifest for click handling on touch devices.
- User settings use `x-icue-property` meta tags (color, combobox, slider, switch, textfield, and others).
- Local Storage is listed as an iCUE reference page in the docs. Use it, wrapped in try/catch, to remember the last station and volume. Do not rely on it for anything important.

## Implementation notes

- Use one `<audio>` element. Switching station sets a new `src` and calls `play()`. Cache-bust is not needed for live streams.
- Handle `waiting`, `stalled`, `error` and `ended`. On error or stall, retry with backoff (2 s, 5 s, 10 s, then every 30 s) while the user has "playing" selected, and show the status.
- Optional: set `navigator.mediaSession` metadata (station name) so Windows media controls and iCUE's Media widget can show it. It may not work in this engine, so treat it as a bonus.
- Settings: default station, autoplay on load (off by default), volume, accent color, optional custom address for the VRT NWS stream.
- Layout: design for 840x344 first. Dark theme. Station buttons are a segmented control; the play button is the biggest element.
- Keep it dependency-free, single `index.html` plus manifest and icon.

## Known risks (test each, report which one fails)

1. **Mixed content.** The Radio 1 address is `http://`. If the widget page is served over HTTPS, the engine may block it. If so, try the `https://` form of the same host and path. TLS on that server is **not verified**.
2. **Autoplay.** Chromium may refuse `play()` without a user gesture. The Play button is a gesture, but autoplay on load may fail. Test it, and say so in the README if it doesn't work.
3. **Page lifecycle.** iCUE may unload a widget when the Edge switches to another page, which would stop the audio. Test with two Edge pages.
4. **Audio routing.** Check what shows in the Windows Volume Mixer (likely iCUE) and that the Windows Volume widget controls it.
5. **Stream formats.** If MP3 stalls, try the AAC stream only after its full address is known.

## Build and test

1. `icuewidget init radio-widget`, add the permissions above, build against the Radio 1 stream first.
2. `icuewidget validate`, `icuewidget package`, import into iCUE (Widgets, `+`), place in a Small slot.
3. Tests on the real display, in this order: plays Radio 1 within a few seconds; switch to VRT NWS and back; volume slider works; pause and resume; unplug the network for 30 seconds and confirm it recovers; leave it for an hour; switch Edge pages and check whether audio continues.

## Acceptance criteria

- Radio 1 plays from the widget on touch, with working play/pause and volume.
- VRT NWS plays once a verified address is configured.
- Network loss recovers by itself and shows a status, never a silent failure.
- Fits a Small slot with no scrolling or clipping.
- A short README says what each setting does and any limit found in the risks above.

## Sources

- Stream addresses: supplied by Christian from VRT. Only the Radio 1 MP3 address was complete.
- Elgato widget docs: https://docs.elgato.com/icue/widgets/specification and https://docs.elgato.com/icue/widgets/references/permissions
- Corsair custom widget guide: https://www.corsair.com/es/ls/explorer/diy-builder/accessories/how-to-create-a-custom-widget-for-the-xeneon-edge/
- VRT on listening to Radio 1 and VRT NWS: https://www.vrt.be/vrtnws/nl/2019/07/01/waar-en-hoe-kan-je-luisteren-naar-vrt-nws/
